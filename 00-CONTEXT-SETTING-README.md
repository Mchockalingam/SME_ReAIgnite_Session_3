
## The situation we are actually in

Sometime in the last eighteen months, most of us stopped writing the majority of our code. We describe, an agent produces, we skim, we merge. The velocity gain has been real and nobody wants to give it back.

What has not changed is the number of hours in a day that a human being can spend carefully reading code.

That is the whole problem in one sentence. Machines can now generate code in parallel, overnight, at weekends, without fatigue. Review capacity is a flat line. Every week, the gap between those two curves widens, and the thing that fills the gap is not review — it is the *appearance* of review. Green checks. A glance at the diff. An approve click.

This session is about what to do about that.

---

## Why your instincts have quietly stopped working

You have spent years building a reliable sense for bad code. Messy formatting means someone was rushing. Odd variable names mean inexperience. A large diff with no tests means corners were cut. Those heuristics work because, for human authors, surface quality tracks care.

Machine written code breaks that link completely. A model produces well formatted, well named, well commented code with comprehensive looking tests, because it is reproducing patterns from high quality training data. The surface is always good. The surface tells you *nothing* about whether the logic underneath is right.

So the failure mode is not "the AI writes bad code."

> **The AI writes code that looks so good you stop scrutinising it.**

Human code fails loudly and looks bad. Machine code fails quietly and looks great. You are no longer scanning for sloppiness. You are scanning for plausible looking wrongness, and that is a different skill.

---

## A worked example, so this is concrete

Here is a real shape of failure. A PR adds request validation to a payments endpoint. Forty-eight lines. Fourteen green checks. Coverage up four points. Clean naming, sensible structure, a docstring.

It imports a function that does not exist. The package renamed that export two major versions ago; the model reproduced the older API from its training data. The tests pass because the test file mocks the module. The reviewer approves in ninety seconds because everything about it reads as competent work.

Three weeks later it is a production incident.

Nobody in that story was careless. A reliable instrument stopped working, and instruments that quietly stop working are more dangerous than instruments that visibly break.

---

## What we will cover

1. **Why review broke** — the throughput asymmetry, and what it does to the four jobs code review has always done: compliance, knowledge sharing, quality, security.
2. **Seven ways machine code fails differently** — hallucinated APIs, tautological tests, cargo-culted patterns, over-abstraction, missing edge cases, confidently wrong business logic, stale dependencies. With live examples.
3. **A framework** — VERDICT, and a five-gate pipeline that makes each check cheaper than the one after it.
4. **The implementation** — actual code you can lift: the grounding checker, the mutation gate, the multi-agent reviewer, and the split between `AGENT.md` and `SKILL.md`.
5. **Tooling** — open source versus commercial, and how to run a bake-off that tells you something a vendor demo cannot.
6. **Rollout and metrics** — ninety-day plan, and the numbers that reveal whether your team is calibrating well or quietly surrendering.
7. **Close** — three things to do this week.

---

## Three ideas to arrive with

**One. Separate the writer from the reviewer.** A coding agent is optimised to produce code. A review harness is optimised to surface risk. Asking one system to do both is asking one objective function to serve two masters. Never let an agent be its own only reviewer.

**Two. Context beats model choice.** The single largest determinant of AI review quality is not which model you use. It is whether the reviewer knows what the change was *supposed* to do. A reviewer with only the diff finds style problems. A reviewer with the diff plus enumerated acceptance criteria finds requirements gaps — which is where the expensive defects live. **Your ticket quality is now a production engineering control.**

**Three. Determinism is the answer to probabilism.** If your definition of "correct" is a paragraph of English, an agent will satisfy the paragraph and you will discover in production what the paragraph did not say. If your definition of correct is a property test, an invariant, a mutation score floor, or a model-checked specification, then the agent does not just write the code you asked for — it writes code that satisfies the specification, and it verifies its own work. Techniques that used to be reserved for elite teams are now cheap. The investment that used to slow you down is now the thing that lets you go fast.

---

## What to bring

**One AI-generated PR from the last month that you merged without reviewing deeply.** Not to shame anyone — everyone in the room has one, including whoever is presenting. We will use them in the closing exercise. Have it open in a tab.

Optionally, be ready to answer three questions about your team:

- Roughly what proportion of merged code is machine-authored now?
- What is your post-merge defect rate, and can you split it by author type?
- When did someone last write a ticket with three or more acceptance criteria?

If you cannot answer the second one, that is itself a finding, and it is where module six starts.

---

## What you will leave with

- A named taxonomy for how machine code fails, so you can spot it in a diff
- A gate-by-gate pipeline design you can apply to your own repository
- Working code: grounding checker, mutation gate, multi-agent reviewer, policy file
- A reference `AGENT.md` and `SKILL.md`, and a clear rule for what belongs in each
- A scored rubric for choosing tools, and a bake-off protocol
- Six metrics that tell you whether your review process is real or ceremonial

---

## One question to hold in your head

Think of the last PR you approved that an agent wrote.

**Did you approve it because you checked it — or because everything was green?**

Be honest with yourself about the answer. That gap is the entire subject of this session, and closing it is neither about working harder nor about trusting the machine less. It is about moving the checks a machine can do reliably onto a machine, so that your attention lands only where it is genuinely irreplaceable: abstraction, edge cases, domain correctness, and the small number of places where the system must never, ever be wrong.

See you in the session.
