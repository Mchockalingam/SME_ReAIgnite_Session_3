# Reviewing What the Machine Wrote — AI Code Review Essentials
### 90-minute session pack | Senior Technical Architect track

---

## What is in this pack

| # | File | Purpose | Audience |
|---|---|---|---|
| 00 | `00-README.md` | This index, source notes, delivery prerequisites | Facilitator |
| 01 | `01-session-guide-90min.md` | Full session content, minute-by-minute, speaker notes, demos, exercises | Facilitator |
| 02 | `02-framework-and-implementation.md` | The **VERDICT** framework + complete technical implementation, all review code | Facilitator + engineers |
| 03 | `03-AGENT.md` | Reference `AGENT.md` / `AGENTS.md` / `CLAUDE.md` file — drop into repo root | Engineers |
| 04 | `04-SKILL.md` | Reference `SKILL.md` for the `ai-generated-code-review` skill | Engineers |
| 05 | `05-tooling-landscape.md` | Open-source tools + GitHub repos; commercial tools + links | Architects, EM, procurement |
| 06 | `06-selection-and-customization.md` | Buy-vs-adopt evaluation scorecard; what to customize if you go open source | Architects, EM, procurement |
| 07 | `07-handouts-and-checklists.md` | Printable reviewer checklist, PR template, one-pager, metrics sheet | All attendees |

Suggested distribution: send `07` before the session, `01`–`06` after.

---

## Source references used to build this pack

| Reference | Status | What was taken from it |
|---|---|---|
| `blog.colinbreck.com/adapting-to-ai-code-review/` (Colin Breck, Mar 2026) | Retrieved in full | Compliance bottleneck, knowledge-sharing erosion, "focus on determinism to define success", multi-model security review, choke points, open-source maintainer strain |
| `tenki.cloud/blog/reviewing-ai-generated-code-checklist` (Eddie Wang, Mar 2026) | Retrieved in full | Seven failure modes of agent-generated code, trust tiers, review template, review-effectiveness metrics, human/AI division of labour |
| `learn.deeplearning.ai/courses/ai-code-review` (DeepLearning.AI × Qodo) | Lesson transcript retrieved | Independent review harness vs. generation harness, pre-PR review, task-level context / ticket quality as the review oracle, context engine, specialized review agents |
| `youtube.com/watch?v=SXg08HPpKr8` — *"AWS Veteran: How The New Software Development Life Cycle Works"* (Heitor Lessa) | **Page returned HTTP 429; transcript not retrievable.** Title and premise confirmed via search: a refactor that consumed ~200M tokens forced a rebuild of the SDLC around agents. | Used only as a framing anecdote in Module 1. **Facilitator should watch before delivery** and swap in their own notes. |
| `youtube.com/watch?v=W1uG25of2t0` | **Page returned HTTP 429; could not be identified.** | Not incorporated. Facilitator should watch and fold in. |

Every factual claim about tools, pricing and benchmarks in `05` and `06` carries a "verify before you quote" flag. This market moves monthly — re-check vendor pricing pages on the day you present.

---

## Prerequisites for delivery

**Facilitator**
- A repo you can demo against with a real AI-generated PR (or use `code/` samples in `02`)
- API key for at least one model provider, exported as `ANTHROPIC_API_KEY`
- `gh` CLI authenticated; Python 3.11+; Node 20+
- One commercial reviewer connected to a sandbox repo (free tier is fine) for the side-by-side in Module 5

**Attendees**
- Laptop optional. Sessions run better if Modules 3 and 5 are watched, not typed.
- Ask each attendee to bring **one AI-generated PR from the last month** they merged without deep review. Module 7 uses these.

---

## Session at a glance

```
00:00  M0  Cold open — the PR that looked perfect                    5 min
00:05  M1  Why review broke: the throughput asymmetry               10 min
00:15  M2  Seven ways machine code fails differently                15 min
00:30  M3  The VERDICT framework and the 5-gate pipeline            20 min
00:50  M4  Implementation: pipeline, agents, AGENT.md vs SKILL.md   20 min
01:10  M5  Tooling: open source vs commercial, how to choose        10 min
01:20  M6  Adoption roadmap and the metrics that keep you honest     7 min
01:27  M7  Close, commitments, Q&A                                   3 min
```

---

## The one-sentence thesis

> Human code fails loudly and looks bad. Machine code fails quietly and looks great.
> So stop reviewing for *sloppiness* and start reviewing for *plausible-looking wrongness* —
> and make as much of that check deterministic and automated as you possibly can.
