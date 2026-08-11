# graph-support

Something in The Graph is broken and nobody is answering. Open an issue.

This is community triage for subgraph developers, indexers, delegators and anyone else
using The Graph. You get a root cause, a workaround, or the name of the party who can
actually fix it. You do not get "I'll raise this internally" and then silence.

Run by **The Night's Watch**, an open community for the people who build and hold the data
layer of web3. We are not Edge & Node, we are not The Graph Foundation, and we have no
authority over either. We are operators and engineers who use this network every day and
got tired of watching questions go unanswered.

### 👉 [Join the Night's Watch Discord](https://discord.gg/CQewvyJ69Y)

File the issue here so the answer is permanent and searchable. Come to the Discord if you
want to talk it through while someone is actually looking at it, if your problem is
urgent, or if you would rather be told in a sentence than wait for a write-up. Indexers,
subgraph developers and delegators are all in there, and on most days that room will
answer you faster than any official channel will.

## What we promise

1. **A human reply.** Target is 48 hours. Not a bot, not a redirect to another channel.
2. **A public disposition.** No issue is closed silently. Every one ends with a stated
   outcome, and the outcome is written down where the next person can find it:

   | Disposition | Meaning |
   | --- | --- |
   | `root cause found` | We know why. The explanation is in the thread. |
   | `fixed` | It works now, and the thread says what changed. |
   | `handed off` | Not ours to fix. We name who owns it and where we told them. |
   | `cannot reproduce` | We tried. Here is exactly what we tried. |
   | `out of scope` | Honest no, with somewhere better to go. |

3. **A fast no.** If your problem is Subgraph Studio, the upgrade indexer, or anything
   else running on Edge & Node's own infrastructure, nobody outside E&N can fix it. We
   will tell you that within a day and point you at them, so you are not also waiting on us.

## What we do not promise

Uptime, an SLA, a fix, or a response to everything. This is unpaid work by a small number
of people. When we cannot get to something we will say so rather than let it rot.

## Before you open an issue

**Never paste an API key, a private key, a mnemonic or an operator keyfile.** Issues here
are public and permanent. A Studio query URL contains your API key: redact it. If you
already posted one, rotate it, do not just edit the message.

Have these ready, because the first thing anyone will ask for is:

- The **deployment ID** (`Qm...`), not just the subgraph ID. They are different things and
  the deployment ID is the one that identifies what is actually running.
- The **chain**.
- The **exact error text**, copied whole. `bad indexers: {...}` with the map intact is
  diagnosable; "it says bad indexers" is not.
- The query, if there is one, and what you expected instead.

## Start here

Some answers already exist. Check before filing:

- [Decoding gateway errors](reference/gateway-errors.md) — what `bad indexers`,
  `BadResponse(400)`, `no attestation: indexing_error` and `too far behind` actually mean,
  verified against gateway source.
- [Who owns what](reference/escalation.md) — which failures E&N alone can fix, which
  belong to an individual indexer, and which are yours.
- [Closed issues](https://github.com/nightswatchhq/graph-support/issues?q=is%3Aissue+is%3Aclosed)
  — every past answer, searchable. This is the point of the repo.

## How we work

[TRIAGE.md](TRIAGE.md) describes the process, the labels and the standards a write-up has
to meet before an issue is closed. It applies to us as much as to anyone who wants to help.

## Helping

Answer someone. That is the whole contribution model. If you are an indexer and an issue
names your address, you are the fastest path to an answer and we would rather have you
than our guess.

The people doing the answering coordinate in the
[Night's Watch Discord](https://discord.gg/CQewvyJ69Y). If you know things about this
ecosystem that are not written down anywhere, which is most of what anybody knows, that is
where to come and be useful. The answer still belongs here afterwards, where it can be
found again.
