---
name: debugging-from-facts
description: Use when debugging a bug, regression, outage, latency spike, memory leak, hang, timeout, or intermittent failure — especially in code nobody on hand fully understands (agent-written, legacy, another team's), when the first explanation came from reading code, when proposing a mitigation or rollback, or when judging someone else's root-cause claim or repro.
---

# Debugging From Facts

## Overview

Code tells you what *can* happen. Facts tell you what *did* happen. Keep two
lists: **facts** (gathered by tools, never changed by you) and **hypotheses**
(everything you infer). Every fact is a constraint. A hypothesis that one fact
contradicts is dead. Debugging is finding the one hypothesis (or the smallest
set of necessary causes) that fits every fact.

If a general debugging skill is also loaded (for example
superpowers:systematic-debugging), use this skill in place of its "read the
code and form a hypothesis" step.

## The rules

1. **Act only on a hypothesis with no ✗.** This includes a mitigation, a
   rollback, and a "safe first step". If you write "this does not fit fact N"
   about a hypothesis, you cannot recommend an action based on it. A rollback is
   an experiment: say which hypothesis predicts that it fixes the bug.
2. **A summary is not a fact.** "v1.8 changed three things", "staging is the
   same", "nothing changed in config" are claims. Get the raw source (the full
   diff between the last good and first bad version, including config, manifests,
   infra, and dependency locks; the live config with `nginx -T`, `env`, the
   real object from the cluster).
3. **Find the control.** Compare the broken case (the variable) with a case that
   differs in as few ways as possible: the same request that succeeded, the
   flag off, the last good version, the healthy replica, the other user. A data
   point that is in the variable but not in the control is a signal. A data
   point in both is noise (it can still kill hypotheses, but it cannot steer).
   If the facts already contain successes and failures, that split is your
   first control.
4. **Read code last.** Use code to explain a strong signal, not to find one.

## The loop

1. **Gather facts, cheapest first:** the bug report, error bodies and headers,
   logs, telemetry, config, deploy and git history. Collect the same fact from
   several instances of the bug when you can.
2. **Isolate signals** with the control (rule 3).
3. **Build the matrix.** Rows = hypotheses. Columns = facts. Cells:
   ✓ consistent, ✗ contradicts, – says nothing, ? unclear.
   - Read each **column**: a fact that almost every row fits is a *weak* signal
     (p99 latency, error rate, "it is slow"). A fact that one row fits is a
     *strong* signal. Steer to the row that the strong signal points to.
   - Read each **row**: one ✗ kills it.
4. **If no row is all ✓**, try these in order, and rebuild the matrix after each:
   1. Recheck how each fact was gathered (wrong host, time window, version, tool?).
   2. Split a hypothesis that bundles several ideas, and keep the part that survives.
   3. Merge many narrow hypotheses (one per symptom) into one shared cause.
   4. Combine two hypotheses that each fit facts the other cannot. Each cause
      must be necessary: remove it and some fact is unexplained.
   5. Only then, a new hypothesis, built from the facts.
5. **If every signal is weak or step 4 fails:** gather facts from one layer down
   (see `layers.md`). Stop as soon as a strong signal appears.
6. **Validate:** check the surviving row against *every* fact, including the
   noise. If several rows survive, you need more facts, not more hypotheses.

## Repros and reviews

- A repro is a hypothesis. Accept it only if it shows the *same signal* as the
  real failure (the same error body, the same frame, the same packet pattern),
  not only the same symptom.
- A review (subagent, codex, a person) must try to **eliminate**: find one fact
  that the hypothesis fails, test whether the signal is also in the control, and
  remove each cause to test that it is necessary. Agreement proves nothing; the
  reviewer cannot see facts that were never gathered.
- Every claim cites a fact. Drop a claim that has no citation.

## Output shape

Report these items, in this order:
1. Facts (numbered, each with its source).
2. The matrix.
3. The verdict: the root cause **or** "not known yet".
4. The next fact to gather, and which rows it will kill.
5. The fix, or the decision for a person. When there are several fixes with
   trade-offs, list them; do not choose a policy question alone.

## Red flags

| Thought | Reality |
|---|---|
| "Roll back X as a safe first step" | Which row predicts that it fixes it? If that row has a ✗, the rollback is a guess. |
| "This may also explain the latency" | Show the fact. "May explain" is not ✓. |
| "The changelog lists 3 changes" | Read the diff. The cause is often the change nobody listed. |
| "The repro gives the same error" | Same symptom ≠ same signal. Compare the bodies. |
| "The reviewer agreed" | Agreement adds no fact. |
| "I read the code and it must be X" | Code says what can happen. Find the fact. |

When a person had to give you a control, a fact, or a pushback, say what it
was. That is a gap in the process: suggest a check, log, or test that would
give it next time.
