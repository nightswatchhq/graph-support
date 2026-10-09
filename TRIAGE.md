# How we triage

This is the process. It binds us as much as anyone who turns up to help.

## Lifecycle

1. **Acknowledge.** Someone human replies. Target 48 hours. If we cannot look at it
   properly yet, the reply says so and says when, rather than going quiet.
2. **Classify.** Label the area and the owning party. See
   [reference/escalation.md](reference/escalation.md). If the owner is not us, say so in
   the thread immediately. A fast, honest "this is the Foundation's and only they can fix it"
   on day one is worth more than a week of us pretending otherwise.
3. **Investigate.** Reproduce where possible. Say what was actually checked and what was
   not. An untested theory is labelled as a theory.
4. **Dispose.** Every issue closes with one of: `root cause found`, `fixed`, `handed off`,
   `cannot reproduce`, `out of scope`. Never a silent close.

## The write-up standard

The issues are the product. A resolved issue that nobody can learn from is half the job.
Before closing, the thread must contain:

- **What actually happened**, at the level of mechanism. Not "it was an indexer issue" but
  which indexer, in what state, and why the gateway behaved as it did.
- **How we know.** The command, the query, the source file, the block number. If a claim
  is inference rather than observation, mark it as inference.
- **What the reporter should do**, concretely.
- **What anyone hitting this in future should do**, which is often different, and is the
  part that makes the archive worth having.
- **Credit**, by name, to whoever worked it out. Frequently that is an indexer or another
  community member and not us. Say so.

Title issues so that a stranger's search finds them. Put the literal error text in the
title. `bad indexers: BadResponse(400) on every allocated indexer` is findable.
`Query not working` is not.

## Standards of honesty

- Do not assert a mechanism you have not checked. "I believe, but have not verified" is a
  complete and respectable sentence.
- A clean result you did not expect is a reason for suspicion, not celebration. Empty data
  rendering as healthy is the single most common way this ecosystem lies to you.
- Never claim credit for someone else's diagnosis.
- Do not answer a question with a product pitch. If our own tooling is genuinely the right
  answer, it can be mentioned **after** the person's actual problem has been addressed in
  their actual stack. Anyone using this repo to advertise will be told to stop.

## Labels

**Area** — what broke.

`area/studio`, `area/gateway`, `area/graph-node`, `area/indexer`, `area/substreams`,
`area/rpc`, `area/horizon`, `area/docs`

**Owner** — who can actually fix it. Set this early, it is the most useful label here.

`owner/foundation`, `owner/indexer`, `owner/upstream`, `owner/reporter`, `owner/watch`

`owner/edge-and-node` is retired. It stays on issues from before the August 2026 handover,
when Edge & Node ran Studio, the Explorer, the upgrade indexer and the gateway. Anything
current on that infrastructure is `owner/foundation`.

**Status**

`status/triage`, `status/investigating`, `status/handed-off`, `status/needs-info`

**Disposition** — set on close.

`root-cause-found`, `fixed`, `handed-off`, `cannot-reproduce`, `out-of-scope`

**Other**

`reference` — issues that exist as durable answers rather than live problems.
`seed` — written up from public threads elsewhere to bootstrap the archive.
`kind/symptom` — a failure mode the Academy's symptom index does not cover yet.

## Feeding the symptom index

The [Academy's symptom index](https://learn-thegraph.com/diagnose/) is indexed by what
somebody is seeing rather than by which part of the protocol owns it, and it only holds
failures somebody has written up. This repository is where those write-ups come from.

**When an issue closes with a failure mode the index does not already carry, open a pull
request against
[`src/lib/diagnostics.ts`](https://github.com/nuthatch-org/graph-academy-v2/blob/main/src/lib/diagnostics.ts)
adding it, and link the issue as a worked case.** Not optional, and not a nice-to-have.
Without it this repository is a pile of threads that happen to be searchable, and the
person hitting the same thing next year has to already know the words we used.

What the index wants is narrower than what a good issue contains:

- **The symptom**, said the way somebody says it out loud while their thing is broken.
- **The causes**, most likely first, with the silent ones ahead of the loud ones.
- **The check that distinguishes each one.** This is the part that carries the weight. A
  cause without a check is a guess with a confident tone.
- **The issue number**, so the mechanism has a worked case attached to it.

Three of the causes in the index today came out of closed issues here rather than from
anybody planning them, which is the argument for the rule. Reports arrive through
[the symptom template](https://github.com/nuthatch-org/graph-support/issues/new?template=06-symptom.yml)
and get triaged like anything else.

## Handing off

When something belongs to the Foundation, an indexer or upstream, we do three things: say so in
the thread, raise it wherever they actually read, and record in the thread where and when
we raised it. Then we label `status/handed-off` and leave it open until there is an
outcome. Handed off is not closed. It is the state where somebody else has the ball and we
are still watching it.

## Going stale

An issue waiting on the reporter for information gets `status/needs-info` and closes after
three weeks of silence, with a note that reopening is welcome. An issue waiting on us does
not close. If we are never going to get to it, we say that out loud and close it
`out-of-scope`, which is at least honest.
