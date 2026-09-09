# Handouts and Checklists
### Print these. Keep them open during reviews.

---

## Handout 1 — The one-page reviewer checklist

> **Mental model:** you are not scanning for sloppiness. You are scanning for
> plausible-looking wrongness. The code will look good. That tells you nothing.

### Before you read the diff

- [ ] What tool and model wrote this? What trust tier?
- [ ] Is there a linked ticket? Does it have **three or more acceptance criteria**?
      If not, you have no oracle — say so in your review and restrict yourself to
      what the code alone can prove.
- [ ] Which choke points does this touch? (auth · money · data governance · crypto
      · destructive migration · public contract)
- [ ] Did the automated gates pass? Do not re-derive what a script already proved.

### The seven checks

**1 · Imports and APIs**
- [ ] Every import resolves against the lockfile
- [ ] Every method exists on the object it is called on, **in the pinned version**
- [ ] Endpoints, config keys, env vars, feature flags and DB columns all exist

**2 · Test efficacy**
- [ ] For each test: *if I broke the implementation, would this fail?*
- [ ] No mock configured to return exactly what the assertion expects
- [ ] No expected value computed by the code path under test
- [ ] Error branches have tests, not just the happy path
- [ ] Every acceptance criterion maps to a named test

**3 · Pattern consistency**
- [ ] Same error handling as the rest of the codebase
- [ ] Same data access, config, logging approach
- [ ] Any new pattern has a stated reason

**4 · Abstraction level**
- [ ] Complexity proportional to the feature
- [ ] Interfaces do not outnumber implementations
- [ ] Rule of three: three real cases, not one real and two imagined

**5 · Edge cases** — trace the unhappy paths *before* the happy path
- [ ] null / undefined on optional fields
- [ ] timeout vs. connection-refused vs. DNS failure
- [ ] empty result vs. error
- [ ] partial failure in batch operations
- [ ] concurrency, races, retry safety, idempotency
- [ ] resource cleanup on the error path

**6 · Business logic** — check against the spec, not against "looks right"
- [ ] Each acceptance criterion: satisfied / violated / not implemented?
- [ ] Order of operations (tax, discount, fee sequencing)
- [ ] Boundary conditions — `>` or `>=`?
- [ ] Units — per second or per minute? cents or pounds?
- [ ] Permission defaults — deny, never allow
- [ ] Any behaviour introduced that no criterion authorises?

**7 · Freshness**
- [ ] New dependencies: last released when? maintained? already in the tree?
- [ ] Framework features used: still the recommended approach?
- [ ] Any deprecation warnings, especially in security-relevant code?

### Before you approve

- [ ] Can I state in one sentence what this change actually does?
- [ ] Would I be comfortable being paged for this at 3am?
- [ ] Am I approving because I checked, or because everything was green?

---

## Handout 2 — The division of labour

Print this and stick it above your desk.

```
MACHINE OWNS THESE — do not spend human attention here
  1  Hallucinated APIs and imports   → grounding checker
  2  Tautological tests               → mutation testing
  3  Cargo-culted patterns            → Semgrep + AGENT.md
  7  Stale / deprecated patterns      → SCA + freshness rules

YOU OWN THESE — this is where your judgment is the only thing that works
  4  Over-abstraction                 → is the complexity proportional?
  5  Missing edge cases               → trace the unhappy paths
  6  Wrong business logic             → check against the spec, not the vibes
```

---

## Handout 3 — Trust tiers

| Tier | You are looking at | Review like |
|---|---|---|
| **T0** | Raw generation. Someone prompted, took the output, saw it compile. | A first-week hire. Check everything. |
| **T1** | Repo-context agent. Indexed the codebase, ran the suite, fixed failures. | A capable mid-level engineer. Focus on logic and edge cases. |
| **T2** | Already scanned by an independent AI reviewer for hallucinations, test quality and convention drift. | A senior peer. Focus on architecture, domain correctness, and whether this change should exist at all. |
| **T3** | Behaviour pinned by property tests, model checking or simulation. | Review the *specification*. The implementation is constrained. |

**If no tier is declared, it is T0.**

---

## Handout 4 — PR template

````markdown
## What and why

Spec: JIRA-____
<!-- No ticket with acceptance criteria = the AI reviewer cannot detect
     requirements gaps. Protected paths will be blocked at Gate 0. -->

## AI provenance (required)

```
AI-Provenance: tool=____; model=____; trust-tier=T_;
               spec=JIRA-____; prompt-ref=.prompts/____.md;
               self-reviewed=____; human-edited=____
```

## Pre-PR review

- [ ] Ran `./review/pre_pr_review.sh` and addressed P1 findings
- [ ] Every new import resolves against the lockfile
- [ ] For each new test: if I broke the implementation, this test would fail
- [ ] Unhappy paths traced: null, timeout, empty result, partial failure, concurrency
- [ ] Implementation checked against the acceptance criteria

## Assumptions I made

<!-- Write them down. An explicit assumption gets reviewed.
     An implicit one ships. -->

## Choke points touched

- [ ] Authorization   - [ ] Money / ledger      - [ ] Data governance
- [ ] Cryptography    - [ ] Destructive migration - [ ] Public contract
````

---

## Handout 5 — Writing a ticket that makes AI review work

Your ticket is now a production engineering control. A thin ticket does not just
make planning fuzzy — it disables requirements-gap detection entirely.

**Thin ticket (produces generic findings):**

> Add under_review status to payments API.
> Should block capture. Add tests.

**Ticket that works (produces requirements-gap findings with evidence):**

> **DX-3 — Prevent capture for payments under manual review**
>
> **Purpose.** Fraud review currently has no way to hold a payment. Payments flagged
> by fraud-service can still be captured by the merchant, which means we settle funds
> on transactions we have already decided are suspicious.
>
> **User story.** As a fraud analyst, I need payments I have flagged to be
> uncapturable until I release them, so that suspicious funds are not settled.
>
> **Acceptance criteria**
> 1. `PaymentStatus` gains an `UNDER_REVIEW` member.
> 2. `POST /payments/{id}/capture` on a payment in `UNDER_REVIEW` returns **409
>    Conflict** with error code `payment_under_review`, and does **not** change status.
> 3. No `payment.captured` event is emitted for a payment in `UNDER_REVIEW`.
> 4. Legal transitions: `AUTHORIZED → UNDER_REVIEW`, `UNDER_REVIEW → AUTHORIZED`,
>    `UNDER_REVIEW → CANCELLED`. All others rejected by the state machine.
> 5. The admin console payment view displays the `UNDER_REVIEW` state.
> 6. Transitions into and out of `UNDER_REVIEW` are written to the audit log with
>    the acting user.
>
> **Out of scope.** Automatic release after a timeout. Bulk release.

Six numbered criteria. Each one is independently checkable against the diff. That
is the difference between a reviewer that comments on docstrings and one that tells
you the endpoint violates AC-2.

**Rule of thumb: fewer than three acceptance criteria means the ticket is not ready.**

---

## Handout 6 — Metrics sheet

Track monthly. Bring the trend to your retro.

| # | Metric | Formula | Healthy | Alarm |
|---|---|---|---|---|
| 1 | Agent-PR first-pass approval | approved with no changes ÷ agent PRs | 40–80% | >90% = not reviewing · <30% = context problem |
| 2 | Post-merge defect rate | defects traced to PR ÷ PRs, split AI vs human | AI ≤ human | AI > human = process gap |
| 3 | Median review time | time from ready-for-review to decision | Falling, with flat metric 2 | Falling **with rising metric 2** = cognitive surrender |
| 4 | Revision categories | count by failure mode | Even spread | One mode dominating = write a rule for it |
| 5 | AI finding acted-on rate | fixed or acknowledged ÷ findings | >40% | <40% = noise; developers are skimming |
| 6 | Choke-point review realness | median time on choke-point reviews | >10 min | <2 min = the control is theatre |
| 7 | Mutation score on changed files | killed ÷ (killed + survived) | ≥ floor, rising | Flat for two quarters = ratchet the floor |
| 8 | Grounding failures caught | count per month | Falling over time | Rising = agent context is degrading |

**The one to watch:** metric 3 crossed with metric 2. Falling review time is only
good news if the defect rate is not rising. If both move at once, your team has
stopped reading and started rubber-stamping — and everything looks fine right up
until it does not.

---

## Handout 7 — Wall poster

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│   HUMAN CODE fails loudly and looks bad.                     │
│   MACHINE CODE fails quietly and looks great.                │
│                                                              │
│   ─────────────────────────────────────────────────────      │
│                                                              │
│   Separate the writer from the reviewer.                     │
│   Context beats model choice.                                │
│   Determinism is the answer to probabilism.                  │
│   Spend human attention only where it is irreplaceable.      │
│                                                              │
│   ─────────────────────────────────────────────────────      │
│                                                              │
│   If everything is green and you have not checked            │
│   the business logic against the spec,                       │
│   you have not reviewed anything.                            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```
