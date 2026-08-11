# Decoding gateway errors

You sent a query to a subgraph on the network and got something like this back:

```json
{"errors":[{"message":"bad indexers: {0x1b7e0068ca1d7929c8c56408d766e1510e54d98d: BadResponse(unattestable response: The skip argument must be between 0 and 20000, but is 21000), 0x326c584e0f0eab1f1f83c93cc6ae1acc0feba0bc: BadResponse(400), 0xedca8740873152ff30a2696add66d1ab41882beb: Unavailable(too far behind), 0xf92f430dd8567b0d466358c79594ab58d919a6d4: BadResponse(no attestation: indexing_error), 0xfeff9093f6b32d0e5cddba743b06a1fedb87c004: Unavailable(no status: indexer not available)}"}]}
```

It looks like noise. It is not. It is a per-indexer report of exactly why every candidate
was rejected, and it usually tells you whether the problem is the network, one specific
operator, or your own query.

**Provenance.** Everything below is read out of the gateway source, `src/errors.rs` and
`src/indexer_client.rs` in `edgeandnode/gateway` v27.6.0. Both files are unmodified in our
fork, so this describes the gateway that actually served your query. It is not folklore.

## How to read it

The map is `indexer address` to `rejection reason`, one entry per indexer that was tried.
Read every entry. The single most common mistake is seeing one scary entry and concluding
the network is down, when four of the six entries say something entirely different.

Three questions, in order:

1. **Do all the entries say the same thing?** If every indexer rejected your query for the
   same reason, the cause is almost certainly your query or the deployment, not the operators.
2. **Do the entries disagree?** Then the deployment is served unevenly. Some operators are
   healthy and something transient or selection-related kept you off them.
3. **Is it `no indexers found` instead?** Different error, different meaning. See below.

## Top-level errors

From `src/errors.rs`. These replace the `bad indexers` map entirely.

| Error | Meaning | Whose problem |
| --- | --- | --- |
| `no indexers found` | Nobody is allocated to this deployment. There was nothing to try. | The subgraph has no indexers. Signal it, or self-host. |
| `bad indexers: {...}` | Indexers exist and were tried. Every one was rejected. | Read the map. |
| `auth error: ...` | API key rejected, out of quota, or not authorised for this subgraph. | Yours, or Studio billing. |
| `bad query: ...` | The GraphQL failed to parse or validate. | Yours. |
| `subgraph not found: ...` | The subgraph or deployment ID does not resolve. | Usually a wrong or unpublished ID. |
| `internal error: ...` | Gateway bug or exceptional condition. | The gateway operator's. |

`no indexers found` and `bad indexers` are frequently confused. The first means zero
allocations. The second means allocations exist but not one produced a usable answer.

## Per-indexer reasons

Each map value is one of three shapes.

### `Timeout`

The request to that indexer timed out. Nothing more is known.

### `Unavailable(<reason>)`

Rejected before a usable response existed. The reasons, from `UnavailableReason`:

| Reason | Meaning |
| --- | --- |
| `too far behind` | The indexer is too far behind chain head to serve an unconstrained query. It is syncing, or stalled. |
| `missing block: N, latest: M` | Your query was pinned to block `N`; that indexer has only reached `M`. |
| `no status: <msg>` | The indexer failed to report its version or indexing progress in time. `no status: indexer not available` is the common form and usually means the indexer-service is down or unreachable. |
| `not supported: <msg>` | The indexer's service version is below the gateway's minimum. Their upgrade, not yours. |
| `blocked (<reason>)` | The gateway itself declined to use this indexer. |
| `internal error: <msg>` | Gateway-side fault while evaluating that indexer. |

### `BadResponse(<detail>)`

The indexer answered and the answer was unusable. This is where the useful detail lives.

| Detail | Meaning |
| --- | --- |
| a bare number, e.g. `400`, `500`, `504` | A non-200 HTTP status from that indexer's indexer-service. **See the warning below.** |
| `failed to connect` | TCP connection to the indexer failed. |
| `no attestation: <errors>` | The indexer returned GraphQL errors and no attestation. The text after the colon is the indexer's own error, verbatim. |
| `no attestation` | No errors, but no attestation either. Malformed for a paid query. |
| `bad attestation: <err>` | An attestation was present and failed verification. Serious: that response could not be proven to come from the allocation it claimed. |
| `unattestable response: <error>` | The indexer returned an error that graph-node misclassifies. **See below.** |
| `response too large: N bytes (max M)` | The response exceeded the gateway's size limit. Ask for less. |
| `missing response` | The payload had no GraphQL response field. |

#### The bare status code is a dead end, and that is a gateway limitation

When an indexer replies with a non-200 status, the gateway logs the response body on its
own side and then throws it away, returning only the status number:

```rust
// src/indexer_client.rs
let status = response.status();
if status != StatusCode::OK {
    if let Ok(body) = response.text().await {
        tracing::info!(status = status.as_u16(), indexer_err_response = body);
    }
    return Err(BadResponse(status.as_u16().to_string()));
}
```

So `BadResponse(400)` means the indexer explained itself and you were not shown the
explanation. The reason exists, in the gateway operator's logs. This is why these errors
feel impossible to act on: they are. Your options are to ask the gateway operator to look
up the log line, or to ask that indexer directly.

#### `no attestation: indexing_error` means that indexer's copy has failed

This is one of the most common and most misread entries. The text after `no attestation:`
is passed through from the indexer's own graph-node. `indexing_error` means the subgraph
deployment has **failed on that indexer** and stopped. It is not a network problem and not
a gateway problem. That operator needs to fix or resync their deployment.

If every entry in the map says `no attestation: indexing_error`, the subgraph itself is
broken and every operator has hit the same deterministic failure. Nothing you do
query-side will help.

#### `unattestable response` is often your query, not the indexer

graph-node returns some errors in a form the gateway cannot attest, so the gateway
suppresses them behind this wrapper. The full list lives in
`src/unattestable_errors.rs` and includes:

- `argument must be between 0 and ...` — pagination out of range, including the `skip` cap
- `query is too expensive`, `query has a depth that exceeds the limit`,
  `Possible solutions are reducing the depth`
- `Query timed out`, `service is overloaded and can not run`
- `is larger than the allowed limit of`
- `the chain was reorganized while executing`
- `Store error:`, `Broken entity found in store:`, `panic processing query:`

The first three groups are **your query**. They will fail on every indexer configured the
same way, which makes it look like the network is broken when in fact your request is
being refused everywhere for the same good reason. Read the text inside the parentheses
before blaming an operator.

The `skip` case specifically: indexers can set `GRAPH_GRAPHQL_MAX_SKIP`, and deep
`skip`-based pagination is at the mercy of whatever each operator chose. The durable fix
is cursor pagination, ordering by `id` and filtering `id_gt` on the last row you saw. It
is also considerably faster than deep skip.

## What this error cannot tell you

An empty result is not an error. An indexer that has diverged and serves `{"data":{"things":[]}}`
looks perfectly healthy to the gateway, attests to it happily, and never appears in a
`bad indexers` map at all. If your data is missing rather than your query failing, this
page is the wrong page. See [escalation.md](escalation.md) and open an issue.
