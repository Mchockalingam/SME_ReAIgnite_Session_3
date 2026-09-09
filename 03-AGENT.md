# `AGENT.md` — reference implementation

> **How to use this file.** Copy the block below to your repository root as `AGENT.md`,
> and symlink `AGENTS.md` and `CLAUDE.md` to it so every agent tool picks it up.
> Then delete everything that is not true of your codebase and add what is.
>
> **The rule that matters:** this file is loaded into context on *every single task*
> in this repository. Keep it between 100 and 250 lines. Everything longer belongs
> in a skill. See the boundary table at the bottom of this document.

---

## The file

```markdown
# AGENT.md — payments-service

Read this before writing, changing or reviewing any code in this repository.
Everything here is a constraint, not a suggestion.

## 1. What this service is

Payment authorization, capture, refund and ledger service for a UK/EU merchant
platform. It is the system of record for money movement. It is PCI-DSS in scope,
it is audited annually against SOC 2 Type II, and every defect in it is a
financial defect.

Upstream:   checkout-api (HTTP), fraud-service (gRPC)
Downstream: ledger-service (events), notification-service (events), Stripe/Adyen (HTTP)
Owns:       payments, payment_attempts, captures, refunds, ledger_entries

## 2. Stack and pinned versions

Python 3.12 · FastAPI 0.115 · SQLAlchemy 2.0 (async) · Alembic · Pydantic v2
PostgreSQL 16 · Redis 7 (idempotency keys only, never a source of truth)
pytest · Hypothesis · testcontainers · mutmut
TypeScript 5.6 · React 19 (admin console only, `web/`)

Version constraints are pinned in `poetry.lock` and `package-lock.json`.
**Never introduce a dependency without checking the lockfile first.** If an API
you remember does not exist in the pinned version, it does not exist. Do not
guess at a symbol name — read the installed package.

## 3. Commands

    make install-dev        install everything including test deps
    make build              build and package
    make test               unit + integration (needs Docker)
    make test-unit          unit only, fast
    make lint               ruff + eslint + mypy --strict
    make semgrep            custom convention rules
    make migrate            apply Alembic migrations
    make review             local pre-PR review (run this before opening a PR)

Never run `alembic downgrade` against anything but a local database.

## 4. Architecture rules

- Layering is enforced: `api/` -> `services/` -> `repositories/` -> `models/`.
  A layer may only import downward. `api/` must never touch `models/` directly.
- All money is `decimal.Decimal` or integer minor units. **Never float. Ever.**
- All monetary values carry an explicit `Currency`. There is no default currency.
- Payment status changes go through `PaymentStateMachine.transition()`, which
  owns the legal transition table. Direct assignment to `payment.status` is
  forbidden and is caught by `semgrep/conventions.yaml`.
- Every write endpoint requires an `Idempotency-Key` header and must be safe to
  retry. Assume every request will arrive twice.
- Domain events are emitted through `events.emit()` inside the same transaction
  as the state change (transactional outbox). Never emit before commit.
- No business logic in `api/`. Handlers validate, delegate, and translate errors.

## 5. Error handling

- Service functions return `Result[T, AppError]`. They do not raise.
- Only `api/` translates `AppError` into an HTTP response, via `error_map.py`.
- Raising past a handler boundary is forbidden (`semgrep: no-raw-raise-past-handler`).
- Never swallow an exception. Never `except Exception: pass`.
- Retries: outbound calls use `tenacity` with jittered exponential backoff and an
  explicit budget. Distinguish timeout, connection-refused and DNS failure — they
  have different retry semantics here.

## 6. Logging and observability

- `structlog` only. No `print`, no bare `logging`.
- Every log line carries `payment_id`, `merchant_id`, `trace_id`.
- **Never log:** PAN, CVV, full cardholder name, raw request bodies on payment
  endpoints, or any value from `models.pii`. Redaction helpers are in `obs/redact.py`.
- Every state transition emits a metric. Every external call emits a span.

## 7. Testing standard

- New behaviour needs a test that would FAIL if the behaviour were removed.
  A test that passes against a stub implementation is not a test.
- Mocks may replace I/O boundaries only. Never mock the unit under test, and
  never configure a mock to return exactly the value the assertion expects.
- Invariants go in `tests/property/` as Hypothesis properties, not example tests.
  Anything in `pricing/`, `ledger/` or `state_machine.py` needs property tests.
- Integration tests use testcontainers against real Postgres. No sqlite substitution.
- Mutation score floor on changed files is 0.65 (`make test-mutation`).

## 8. Forbidden patterns

| Do not | Because |
|---|---|
| `float` for money | Rounding errors become financial errors |
| `payment.status = X` | Bypasses the state machine's guards |
| `datetime.now()` | Use `clock.now()` — tests must control time |
| `except Exception: pass` | Silent failure in a money path |
| Raw SQL string interpolation | Injection; use SQLAlchemy expressions |
| New ORM relationship with lazy loading | N+1 under load; use explicit joins |
| `moment.js`, `request`, `pytz` | Deprecated. Use Temporal/Intl, httpx, zoneinfo |
| Adding an abstraction for one call site | Rule of three. Three real cases or none |
| `TODO` without a ticket reference | Untracked debt |

## 9. Security invariants

- Authorization is checked in `services/`, never only in `api/`.
- Permission functions default to **deny**. A `return True` fallthrough is a bug.
- Secrets come from the secret manager at runtime. Never from env files in the
  repo, never hardcoded, never in a test fixture.
- All external input is validated by a Pydantic model at the boundary.
- Cryptographic operations use `crypto/` helpers only. Never call a primitive
  directly, never invent a scheme, never choose your own IV.

## 10. Choke points — stop and involve a human

Do not merge changes to these without a named human reviewer, regardless of how
confident you are or how green the pipeline is:

- `src/auth/**`, anything named `permissions*` or `authorize*`
- `src/payments/**`, `src/ledger/**`, `src/pricing/**`, `src/tax/**`
- `src/crypto/**`, anything named `keys*` or `secrets*`
- `migrations/**` containing DROP, TRUNCATE, ALTER COLUMN or DELETE FROM
- `openapi/**`, `proto/**`, `events/schemas/**` — public contracts
- Anything changing IAM policy, network ACLs or secret configuration

## 11. When you are uncertain — stop

Do not guess. Stop and ask, in the PR description or in chat, when:

- The ticket has no acceptance criteria, or fewer than three
- Implementing the requirement requires assuming a business rule not written down
- The change would alter a public contract or an event schema
- You cannot find an existing pattern in this repo for what you are doing
- A dependency you want does not already exist in the lockfile
- You are about to write a state transition not in the transition table

Writing "I assumed X" in the PR description is always better than silently
assuming X. An explicit assumption gets reviewed. An implicit one ships.

## 12. Definition of done

- Gate 1 green: build, types, lint, semgrep, SCA, secrets, tests, mutation floor
- Every acceptance criterion in the ticket maps to a named test
- Unhappy paths covered: null, timeout, empty result, partial failure, concurrency
- No new `TODO` without a ticket
- PR carries the `AI-Provenance` trailer and a spec link
- `./review/pre_pr_review.sh` run and its P1 findings addressed

## 13. Where things live

- Specs and acceptance criteria: Jira, project DX. Link the ticket in every PR.
- Prompts used to generate code: `.prompts/<TICKET>.md`, committed with the change
- Architecture decisions: `docs/adr/`
- Incident history: `docs/incidents/index.yaml` — read this before changing a
  module that appears in it
- Review policy: `review/policy.yaml` — changing review behaviour is itself a PR
- Review procedure: `.skills/ai-generated-code-review/SKILL.md`
```

---

## What goes in `AGENT.md` — and what does not

### The boundary

`AGENT.md` answers **"what is always true in this repository?"**
`SKILL.md` answers **"how do I perform this specific task?"**

| Signal | Belongs in |
|---|---|
| It is true regardless of what task the agent is doing | `AGENT.md` |
| It is only relevant when performing one particular kind of work | `SKILL.md` |
| It is a fact, constraint or prohibition | `AGENT.md` |
| It is a numbered procedure | `SKILL.md` |
| It changes when the architecture changes | `AGENT.md` |
| It changes when you tune the process | `SKILL.md` |
| It is under ~15 lines and universally relevant | `AGENT.md` |
| It is 200 lines of rubric and worked examples | `SKILL.md` |

### The cost argument

`AGENT.md` is loaded on every task. A 900-line `AGENT.md` costs you context on
every single run, crowds out the actual code being worked on, and — the real
problem — gets skimmed rather than followed, because the important constraints
are buried in procedural detail nobody needed for this task.

A skill costs nothing until its description matches the task at hand. That
asymmetry is the whole reason both file types exist. Every line you move from
`AGENT.md` into a skill makes the remaining lines more likely to be obeyed.

### Concrete split for review work

| Content | File | Why |
|---|---|---|
| "Money is `Decimal`, never float" | `AGENT.md` | True while writing, reviewing, refactoring, debugging |
| "Payment status changes go through the state machine" | `AGENT.md` | Universal constraint |
| The choke-point list | `AGENT.md` | Must be known before *any* change is proposed |
| Build and test commands | `AGENT.md` | Needed by every task |
| "When uncertain, stop and ask" | `AGENT.md` | Behavioural, always applies |
| The seven agent failure modes | `SKILL.md` | Only relevant when reviewing |
| Severity taxonomy and confidence rules | `SKILL.md` | Only relevant when producing findings |
| Step-by-step review procedure | `SKILL.md` | A procedure, by definition |
| How to invoke `grounding_check.py` | `SKILL.md` | Task-specific tooling |
| Worked examples of good and bad findings | `SKILL.md` | Long; only useful in context |
| Finding output JSON schema | `SKILL.md` reference file | Long; loaded only when emitting findings |

### Six anti-patterns to avoid

1. **The kitchen sink.** Everything anyone ever wanted the agent to know, in one
   file. It stops being read. Split it.
2. **Unenforceable prose.** "Write clean, maintainable code" changes nothing.
   "All handlers return `Result[T, AppError]`; see `semgrep/no-raw-raise.yaml`"
   changes behaviour and is mechanically checkable.
3. **Duplication across the two files.** The moment your `SKILL.md` restates your
   tech stack, the two files begin to drift and one of them becomes a lie. Skills
   should assume `AGENT.md` is loaded and reference it.
4. **Documenting the obvious.** "Use meaningful variable names" wastes context
   the model does not need. Document what is *surprising* about your codebase.
5. **Never updating it.** When a convention changes, the PR that changes it must
   change `AGENT.md`. These files are code. Review them like code.
6. **No "stop" conditions.** An `AGENT.md` that tells an agent what to do but
   never what to refuse produces confident work on ambiguous requirements —
   which is exactly failure mode 6. Section 11 is not optional.
