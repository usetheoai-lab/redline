# redline

**An agent that explains why CI went red.** Built with [`@theokit/sdk`](https://www.npmjs.com/package/@theokit/sdk).

When a pipeline fails, `redline` takes the logs, the diff and the repository,
reproduces the failure where it can, and answers **why it broke** — with evidence and
a next step. Not a summary of the log: an explanation.

---

## Why this exists

A red run is the **only** moment the explanation exists. It is destroyed routinely, and
always by the same mechanism: the filter is written for the case you expect, and the
case you expect is green.

Three episodes measured within seventeen hours on one team's repositories:

| what was run | what survived |
|---|---|
| `pnpm test \| grep` | `11 failed`. No test names. The next two runs passed — the explanation ceased to exist. |
| `pnpm gates >/dev/null 2>&1 && echo OK` | The entire output discarded. The `echo` as the only oracle. |
| twenty commits against a green local gate | CI was red the whole time. An outside observer surfaced it. |

None of the three was carelessness. **All three were the command written before the
result mattered.** That is the gap `redline` fills.

---

## This project is built with `@theokit/sdk`

`redline` is an agent, and the harness underneath it is `@theokit/sdk`. Every capability
below comes from the SDK rather than from bespoke code here — that is a deliberate
constraint, not an accident of convenience.

| capability | why this product needs it |
|---|---|
| `buildRepoMap` | understand a repository it does not know |
| `compactTranscript` | **a CI log does not fit the window** — and the useful information is three lines |
| `sandbox` | reproduce rather than trust what the log claims |
| `subagents` | one per failing job, in parallel |
| `persistence` | *"have we seen this failure before?"* — recurrence is the most valuable signal |
| `isTransientError` | a network mirror going down and a broken test demand **opposite** answers |
| `subscription` | stream the analysis while it happens |
| `server/auth` | an authenticated webhook — this is a service, not a script |
| `Eval` + `Scorers` | was the explanation right? Without this it is a guess delivered with confidence |

**Two of them decide whether the product exists.** `compactTranscript`, because if
compaction drops the three lines that matter there is no explanation to give. And
`isTransientError`, because calling both an unavailable `apt` mirror and a broken test
"a CI failure" is getting precisely the thing the product sells wrong — one network
step hung a PR for 1h51 while a genuinely broken test stayed red for four hours.

If a capability is missing, awkward, or undocumented, **that is a finding**. Building
this product is how we learn what the SDK is actually like to use.

---

## Inviolable rule of this repository

> This project consumes `@theokit/sdk` **from the npm registry only**.
>
> No `file:`, no `pnpm link`, no `workspace:`, no reaching into a local checkout.
> **If something only works with the source at hand, that is an issue — not a workaround.**

The reason is not purism. This repository exists to measure what a developer experiences
installing the SDK without knowing it from the inside. The moment it consumes local
source it stops measuring what is published and starts measuring what exists on one
machine.

The same discipline applies to knowledge: **whoever builds here should not read the
SDK's source.** Documentation, published types and error messages are the whole surface.
Routing around a problem because you know the internals is losing the measurement.

---

## Status

**Day 0.** Nothing implemented. Open decisions:

- [ ] **First repository watched.** It should be a project this codebase has no inside
      knowledge of — otherwise the agent gets credit for what the author already knew,
      and the repository map, the log shapes and the failure modes stop being discoveries.
- [ ] Comments on the PR autonomously, or waits for human approval
- [ ] Entry surface: webhook, GitHub App, or polling

---

## How we report

Every friction becomes an issue, **documentation friction included**. *"The README did
not say"* is a legitimate issue and the most under-reported one, because whoever trips
on it solves it alone and moves on. For adoption, it is the most expensive.

Issues about the SDK go to the SDK's repository, carrying: what was attempted, what was
expected, what happened, and the exact installed version.
