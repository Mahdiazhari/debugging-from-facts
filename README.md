# debugging-from-facts

A skill for coding agents (Claude Code, Codex, and others that read `SKILL.md`).
It makes an agent debug from facts, not from a story it built by reading code.

Use it when an agent debugs code that nobody on hand fully understands:
code that an agent wrote, legacy code, or another team's code.

## The idea

Code tells you what *can* happen. Facts tell you what *did* happen.

- Keep two lists: **facts** (from logs, error bodies, configs, diffs, profiles)
  and **hypotheses** (everything you infer).
- Every fact is a constraint. A hypothesis that one fact contradicts is dead.
- Compare the broken case with a **control** that works. A difference is a
  signal. Data that is in both is noise.
- Put hypotheses and facts in a matrix. A fact that almost every hypothesis
  fits is a weak signal. A fact that only one fits is a strong signal.
- When no hypothesis fits, collect facts from one layer further down. Do not
  invent more hypotheses.
- Accept a repro only when it shows the same *signal* as the real bug, not only
  the same symptom.

The full method is in
[`skills/debugging-from-facts/SKILL.md`](skills/debugging-from-facts/SKILL.md).
What to collect at each layer is in
[`layers.md`](skills/debugging-from-facts/layers.md).

## Does it work? Partly. Read this before you rely on it.

We tested it the way you test code: without the skill, then with it. All
tests use Claude (October 2026), 3 runs per case, graded by an LLM judge.
The cases are in [`evals/`](evals/) and [`evals-invoked/`](evals-invoked/).

**1. The scenario the skill was written from** (`evals/*-upload-413`: uploads
fail with HTTP 413 after a release). This is not a fair test, because the
skill was written to fix the failures we saw on it.

| | Without | With |
|---|---|---|
| Thin facts: recommends a rollback that its own facts contradict | 6 of 6 runs | 0 of 3 |
| Thin facts: asks for the full release diff, not the changelog | 0 of 6 | 3 of 3 |
| Thin facts, eval score | 0.25 | 1.00 |
| Full facts, eval score | 1.00 | 1.00 |

**2. Five held-out bugs the skill never saw** (`evals/heldout-*`): a memory
leak, an auth redirect loop, Postgres latency, and two anonymized real
incidents. A separate agent wrote three of them without seeing the skill.

| | Without | With |
|---|---|---|
| Full facts (5 cases), skill loads by itself | 0.98 | 0.98 |
| Full facts (5 cases), prompt asks to use the skill | 1.00 | 0.96 |
| Thin facts (5 cases), skill loads by itself | 0.97 | 0.93 |
| Thin facts (5 cases), prompt asks to use the skill | 1.00 | 0.93 |
| How often the skill loaded by itself | — | about 4 to 7 of 30 runs |

On new bugs, the skill did **not** improve the answers. The "with" scores are
even a little lower. We read all 7 failed "with" answers. 5 are grader or
judge errors (for example, two correct fixes for the logout bug failed a
haiku judge). 1 claimed a correct but unproven cause as certain. 1 is
borderline. None named a wrong root cause. So we count the result as no
difference, not as harm. The agent without the skill was already good on
these bugs. The failure that the skill fixes best (acting on a theory that your own
facts contradict, when the facts are thin and the change list is misleading)
did not appear in them.

Honest summary: the skill gives the agent a clear structure (facts, matrix,
next check) and blocks one known failure. Do not expect it to make an agent
find more root causes. It rarely loads by itself, so call it on purpose:
`/debugging-from-facts`.

Disclosure: after the first held-out run, we rewrote the five "thin" graders.
The first version failed answers that named the true cause as an unconfirmed
best guess and proposed a low-risk change. The new graders apply to both arms.
Some thin graders are still strict (for example, one counts a low-risk
`SameSite` revert as a fix aimed at a distractor).

Run the evals yourself; see below. Results vary between runs.

## Install

### Claude Code, as a plugin

```
/plugin marketplace add Mahdiazhari/debugging-from-facts
/plugin install debugging-from-facts@debugging-from-facts
```

Restart Claude Code. The skill loads by itself when you debug.

### Any agent, as a plain skill folder

```bash
git clone https://github.com/Mahdiazhari/debugging-from-facts.git
# Claude Code, for all your projects:
cp -R debugging-from-facts/skills/debugging-from-facts ~/.claude/skills/
# Codex, Gemini CLI, Copilot CLI:
cp -R debugging-from-facts/skills/debugging-from-facts ~/.agents/skills/
# Or for one project only: copy it into <project>/.claude/skills/
```

## Run the evals

```bash
git clone https://github.com/Mahdiazhari/debugging-from-facts.git
cd debugging-from-facts
claude plugin eval .                            # cases in evals/
claude plugin eval . --eval-dir evals-invoked   # held-out cases, skill requested in the prompt
```

Each command runs every case with the plugin and without it, and reports the
difference. `evals/HELDOUT-ANSWERS.md` has the true root causes.
It uses your own Claude credentials, so it costs tokens.

## Credit

The method comes from Cindy Sridharan's article
[How to effectively debug when you no longer understand the code](https://copyconstruct.medium.com/how-to-effectively-debug-when-you-no-longer-understand-the-code-bd3dcb2c5f7b)
(October 2026). The skill is a short, agent-facing version of her framework:
the factual and derivative tiers, the control, the hypotheses-by-fact matrix,
the five tactics, progressive signal discovery, and repro signal parity. Read
the article for the reasoning and the three real bugs behind it. Mistakes in
this summary are ours.

## License

[MIT](LICENSE)
