# Choosing and Customizing AI Code Review Tooling

Two questions, answered in order:

1. **§1–§3** — What should a technical team look at when deciding whether to buy a
   commercial AI code review tool or adopt an open-source one?
2. **§4** — If the choice is open source, what must be customized to fit the project?

---

# §1 — The evaluation scorecard

Score every candidate 1–5 on each criterion, multiply by the weight, and sum. The
weights below are a starting point for a mid-size regulated engineering org —
adjust them, but adjust them **before** you see any vendor demo, not after.

## 1.1 Context and capability (weight 25)

| Criterion | What to actually test |
|---|---|
| **Context depth** | Does it read only the diff, the repo, or across repos? Give it a PR whose bug is only visible from a caller in another service. Diff-scoped tools cannot pass this and no amount of prompt tuning fixes it. |
| **Cross-file and cross-service reasoning** | Does it detect a breaking change for a downstream consumer? |
| **Monorepo and polyglot handling** | Test on your largest repo, not a sample. Ask about indexing time and cost on first run and on incremental updates. |
| **Spec / ticket integration** | Can it pull the linked Jira/Linear/GitHub issue and evaluate the diff against enumerated acceptance criteria? **This is the single biggest differentiator for catching wrong business logic**, and many tools do not do it at all. |
| **Test-quality analysis** | Does it detect tautological tests, or only missing tests? |
| **Custom rule engine** | Can you encode *your* conventions, or only accept the vendor's opinions? |
| **IDE / pre-PR support** | Can developers run it locally before opening a PR? |

## 1.2 Signal quality (weight 25)

| Criterion | What to actually test |
|---|---|
| **Catch rate on your defects** | Replay real PRs that caused real incidents. See §2. |
| **False-positive rate** | Count findings a senior engineer dismisses. This is the number that determines whether the tool is still switched on in six months. |
| **Comments per PR** | More than ~10 on a normal PR and developers will skim. |
| **Severity calibration** | Is the tool's "critical" your critical? Miscalibration is worse than no severity at all. |
| **Determinism** | Run the same PR three times. How much does the output vary? Non-determinism makes audit evidence weak and makes suppressions unreliable. |
| **Explanation quality** | Does a finding tell you *why*, with evidence, or just assert? |
| **Suppression and feedback** | Can you dismiss a finding permanently? Does dismissal actually change future behaviour? |

## 1.3 Security, data and compliance (weight 20)

| Criterion | What to actually ask |
|---|---|
| **Deployment model** | SaaS / single-tenant / on-prem / air-gapped. If you have a residency constraint, this filter eliminates most of the market immediately. |
| **Training on your code** | Get it in the contract, not the marketing page. |
| **Data retention and residency** | Where is the index stored? For how long? Which region? |
| **Sub-processors** | Which model providers does the tool route to, and can you pin or restrict them? |
| **Model pinning** | Can you fix the model version? Without this, your attestation says "reviewed by something" and your review results silently change under you. |
| **Multi-vendor capability** | Can you run security review across two vendors? A single model's blind spots become your blind spots. |
| **Prompt-injection posture** | Ask directly: what stops a malicious PR from instructing the reviewer? A vendor without a crisp answer has not thought about it. |
| **Permission scope** | What repo permissions does it demand? A reviewer needs read and comment. Anything asking for write access to contents or workflows is a supply-chain risk. |
| **Audit trail** | Can you export per-PR evidence: model, version, findings, decision, approvers? |
| **SSO, SCIM, RBAC** | Table stakes at enterprise scale. |
| **Certifications** | SOC 2 Type II, ISO 27001, and whatever your regulator names. |

## 1.4 Integration and operations (weight 15)

| Criterion | Notes |
|---|---|
| **SCM coverage** | GitHub / GitLab / Bitbucket / Azure DevOps. Multi-platform shops narrow fast. |
| **CI integration** | Can it block a merge? Through required status checks, or only comments? |
| **Latency** | Time-to-first-comment on a 500-line diff. Over five minutes and it falls outside the developer's attention window. |
| **Large-diff behaviour** | What happens at 5,000 lines? Truncate, chunk, or fail? Ask, then test. |
| **API and webhooks** | Can you export findings into your own metrics store? If you cannot measure it you cannot tune it. |
| **Configuration as code** | Is the policy a file in the repo, or clicks in a web console? Console config is unreviewable and undiffable. |
| **Rate limits** | Especially relevant when agents are opening PRs around the clock. |

## 1.5 Commercial (weight 15)

| Criterion | The trap |
|---|---|
| **Pricing model vs. your shape** | Per-seat pricing punishes large teams with few PRs. Per-review pricing punishes agent-heavy workflows — and agent-heavy is exactly where you are heading. **Model your cost at 3× current PR volume, not today's.** |
| **Hidden caps** | Daily review limits, credit allowances, overage rates, "premium request" pools shared with other features. |
| **Total cost including tokens** | Self-hosted OSS is licence-free and token-expensive. Include the API bill. |
| **Exit cost** | Custom rules, suppressions and tuning are the real lock-in, not the contract. Can you export them? |
| **Vendor viability** | This category is consolidating (Graphite → Cursor, Dec 2025). What is your plan if your vendor is acquired or shut down? |
| **Roadmap alignment** | Is the vendor a review specialist, or is review a bundled feature of something else? Bundled review tends to stay shallow. |

---

# §2 — The bake-off protocol

**Do not buy anything on a demo.** Vendor demos run on repositories chosen to make
the tool look good. Run this instead. It takes about a week and it is the only
evidence that matters.

### Build the corpus (one day)

1. Select **50 merged PRs** from the last six months of your own repo.
2. Include **15–20 PRs that caused a real defect** — production incident, hotfix,
   revert, or a bug ticket traced back to that PR. These are your ground truth.
3. Include **5 PRs you deliberately seed** with each of the seven failure modes:
   hallucinated import, tautological test, cargo-culted pattern, over-abstraction,
   missing edge case, wrong business logic, deprecated dependency.
4. Include **10 clean PRs** that were genuinely fine. These measure noise.
5. Record which PRs had good tickets and which had thin ones. You want to know how
   much each tool's performance depends on spec quality.

### Run it (two days)

Run every candidate — including your own PR-Agent build — across all 50, on a
replica repo so nobody is influenced by live comments. Same PRs, same order.

### Score it (one day)

| Metric | How |
|---|---|
| **True-positive rate on known defects** | Did it flag the actual cause of the incident, or something adjacent? Only the actual cause counts. |
| **Seeded-defect catch rate, by failure mode** | Tells you which of the seven modes each tool covers. This is more useful than an aggregate score. |
| **False positives per clean PR** | Have a senior engineer classify. Anything they would dismiss is a false positive. |
| **Comments per PR** | Median and 95th percentile. |
| **Spec sensitivity** | Catch rate on good-ticket PRs minus catch rate on thin-ticket PRs. A large gap means the tool genuinely uses the spec — which is a *good* sign, and tells you to fix your tickets. |
| **Time to first comment** | Median and worst case. |
| **Cost per PR** | Actual, including tokens. |
| **Determinism** | Re-run 10 PRs. Measure finding overlap between runs. |

### The decision rule

> The best tool is the one with the **highest catch rate on defects your team
> actually shipped**, at a false-positive rate your team will still tolerate in
> six months. Not the highest raw catch rate. Muting is a real outcome and it
> takes the catch rate to zero.

### The trap to avoid

Do not run the bake-off against a tool's default configuration and then compare it
to your own heavily-tuned pipeline, or vice versa. Give every candidate the same
`AGENT.md`, the same conventions and one day of tuning. Otherwise you are measuring
your own effort, not the tool.

---

# §3 — Buy, build, or both

Most mature teams end up with a hybrid, and this is the right answer more often
than either extreme.

| Layer | Recommendation | Why |
|---|---|---|
| Deterministic analysis (lint, SAST, SCA, secrets) | **Open source, always** | Free, reproducible, no data egress, no vendor risk. There is no commercial argument here. |
| Mutation and property testing | **Open source** | Mature tooling, entirely local |
| Grounding / hallucination detection | **Build** | ~200 lines against your own lockfile. No vendor does this as well as you can for your own stack. |
| Context engine (spec fetch, conventions, ownership, incidents) | **Build** | This is where your domain lives. It is also the highest-leverage code you will write. |
| Semantic LLM review | **Buy, usually** | Repo indexing at scale is genuinely hard engineering. Unless you have a platform team that will own it as a product, buy it. |
| Policy, routing and merge decisions | **Build, always** | Never outsource your merge policy to a vendor's opinion. Keep it in `policy.yaml`. |
| Attestation and audit evidence | **Build** | Your compliance obligations are yours; the format must match your auditor. |

### When to build the semantic layer too

- Hard air-gap or data-residency constraint that no vendor's on-prem offering meets
- A dedicated platform team with capacity to own it as a product, not a side project
- Domain-specific correctness that is your actual differentiator (medical device,
  avionics, trading, safety-critical control)
- PR volume high enough that per-review pricing exceeds an engineer's salary

### When buying is obviously right

- Team under ~30 engineers
- No dedicated platform or developer-productivity function
- Standard stack with strong vendor coverage
- You need something working this quarter

**The thing you should never outsource** is the `AGENT.md`, the conventions, the
rules and the policy. Those encode what your organisation knows. Whichever engine
reads them, they are yours and they are the durable asset — the engine is
replaceable, the encoded knowledge is not.

---

# §4 — If you choose open source: what to customize

Deploying PR-Agent unmodified gets you generic review comments and a token bill.
Roughly **80% of the value is in customization**, and it falls into fourteen areas.
They are ordered by return on effort.

---

## 4.1 Context assembly — *highest leverage, do this first*

Stock tools send the diff. That is why stock output is generic.

**Customize:**
- Pull the linked ticket and, critically, its **enumerated acceptance criteria**.
  Without this, requirements-gap detection is impossible, and requirements gaps
  are where the expensive defects live.
- Include full changed-file contents, not just hunks — models reason far better
  with surrounding code.
- Add a resolved symbol table for touched modules so the reviewer can distinguish
  an invented symbol from a real one without guessing.
- Add pinned dependency versions from the lockfile.
- Add `CODEOWNERS`, relevant ADRs, and **past incidents in the touched paths**.
  Incident history is cheap to wire up and makes edge-case detection markedly better.

**Effort:** 2–3 days. **Impact:** the largest single quality jump available.

## 4.2 Prompts and reviewer personas

Stock prompts are written to be inoffensive across every codebase on earth. Yours
should be opinionated about *your* codebase.

**Customize:**
- Split one generalist reviewer into narrow specialists (correctness, security,
  spec-conformance, test-efficacy, architecture-fit). Narrow rubrics produce fewer,
  better findings.
- Add explicit **out-of-scope** sections to each. The main cause of noise is agents
  reporting things another agent already owns.
- Add confidence discipline: "if you have not seen the definition of a called
  function, cap confidence at 0.6", "an empty findings list is a valid review".
- Add your domain vocabulary. A payments reviewer that does not know what
  "capture", "authorization" and "settlement" mean in your system is guessing.
- Version and **regression-test** your prompts (Promptfoo). They are production code.

**Effort:** 1 week initially, ongoing tuning. **Impact:** very high.

## 4.3 Custom rule packs — moving work out of the LLM

Every review comment you make twice should become a rule the third time.

**Customize:**
- Semgrep rules for your conventions: layering violations, forbidden primitives,
  error-contract enforcement, deprecated internal APIs, state-machine bypasses.
- Rules for your framework's specific footguns.
- Rules for the patterns your models keep getting wrong — you will discover these
  from the revision-request category metric.

Every rule you write is deterministic, free, instant and reproducible, and removes
a class of finding from the probabilistic layer forever.

**Effort:** ongoing, ~1 rule per week. **Impact:** compounding.

## 4.4 Grounding and verification

No open-source reviewer ships this. Build it.

**Customize:** symbol resolution against your installed environment; endpoint
validation against your OpenAPI specs; config-key and env-var validation against
your schema; feature-flag name validation; database column validation against
your migrations.

**Effort:** 2–3 days per language. **Impact:** eliminates the most common failure mode entirely.

## 4.5 Output contract and noise control

**Customize:**
- Enforce a JSON schema on findings. Free-text review cannot be gated, deduplicated
  or measured.
- Confidence floor (start ~0.7), per-PR finding cap (~25), fingerprint-based dedupe.
- A `suppressions.yaml` in the repo, requiring a written justification per entry.
  Suppressions in a web console are invisible and rot; suppressions in the repo get
  reviewed.
- Fold multi-agent output into one sticky comment, not N comments.

**Effort:** 2–3 days. **Impact:** determines whether developers read the output at all.

## 4.6 Model routing, cost and determinism

**Customize:**
- `temperature=0` everywhere. Determinism where you can get it.
- Pin model versions; record the pin in the attestation.
- Route by task: a cheap fast model for summarisation, an expensive one for
  correctness and security.
- Two vendors on the security path, with a confidence penalty for single-vendor
  findings and escalation on agreement.
- Prompt caching for the stable parts of the context pack (`AGENT.md`, conventions).
- Token budget per PR with a hard ceiling. Agent-authored PRs arrive at 3am in
  volume and an unbounded review pipeline is an unbounded bill.
- Skip review entirely on generated files, lockfiles, vendored code and pure
  formatting changes.

**Effort:** 3–4 days. **Impact:** high on cost, moderate on quality.

## 4.7 Risk routing and choke points

Stock tools comment on everything equally. Yours should not.

**Customize:** the choke-point path list, minimum reviewer counts per class,
`CODEOWNERS` integration, automatic reviewer assignment, escalation rules for
critical findings and large diffs, and trust-tier-driven review depth.

**Effort:** 2 days. **Impact:** this is what actually protects human attention.

## 4.8 Merge policy — keep it in your code

**Customize:** which severities block; whether blocking differs by path and by
trust tier; the override path and who can use it (with a recorded reason);
the difference between advisory and required checks.

**Never let the model make this decision.** A finding is input. The decision is
made by deterministic code reading a policy file. This is also your primary
structural defence against prompt injection: nothing written into a diff can
influence code that never treats it as instructions.

**Effort:** 2 days. **Impact:** high, and it is a security control.

## 4.9 Security hardening

**Customize:**
- Run the review job with least privilege: read + PR comment. No `contents: write`,
  no deploy credentials, no workflow-trigger permission.
- Treat the diff, PR body, comments and fixtures as untrusted input. Wrap them with
  an explicit boundary marker in the prompt.
- Add a `prompt_injection` finding category so injection attempts are *reported*
  rather than silently obeyed.
- Redact secrets before anything is sent to a model provider.
- Run review in a network-isolated job where feasible.
- For high-assurance evaluations, pin a static model in a hermetic environment so
  the reviewer cannot reach out and be influenced by a compromised service.

**Effort:** 3–4 days. **Impact:** critical, and frequently skipped.

## 4.10 Language and repository scoping

**Customize:** per-language handling; monorepo path filters so a change in one
package does not trigger review of the whole tree; diff chunking strategy for very
large PRs; exclusion lists for generated code; incremental indexing so first-run
cost is paid once.

**Effort:** varies hugely with repo shape. **Impact:** high in monorepos.

## 4.11 Test-efficacy gates

**Customize:** mutation testing scoped to changed files only (whole-repo mutation
is too slow for CI); a score floor you can actually pass today, ratcheted quarterly;
per-directory floors — higher for money and auth paths; property-test requirements
on modules with invariants.

**Effort:** 3–5 days. **Impact:** permanently eliminates failure mode 2.

## 4.12 Telemetry and the feedback loop

Without this you are tuning blind, and this is the area teams most often skip.

**Customize:** persist every finding with its outcome (fixed / dismissed /
ignored); tag dismissals by reason; track acted-on rate per agent, per category
and per rule; track post-merge defects back to the PR and check whether the
reviewer had flagged it; review the numbers weekly for the first quarter and
monthly after.

**Effort:** 1 week. **Impact:** it is the only way anything else improves.

## 4.13 Attestation and audit evidence

**Customize:** the evidence bundle format your auditor will accept; model and
prompt version pinning; retention aligned to your regulatory period; signing;
and an export path for audit requests.

**Effort:** 3–4 days. **Impact:** low day-to-day, decisive at audit time.

## 4.14 Fork and maintenance strategy

The one people forget until it hurts.

**Customize:**
- Prefer **configuration and extension points over patching the tool**. Every patch
  is an upgrade you will fight.
- If you must fork, keep changes in clearly separated modules with a documented
  rebase procedure.
- Pin the upstream version. Upgrade deliberately, with the bake-off corpus as a
  regression suite.
- Name an owner. An unowned review pipeline degrades silently, and the failure mode
  is that it keeps producing plausible output while catching less — which is
  exactly the problem you adopted it to solve.
- Watch upstream licence changes. Several projects in this space hold enterprise
  features behind commercial keys, and that boundary moves.

**Effort:** ongoing. **Impact:** determines whether this still works in two years.

---

## §4.15 Realistic effort summary

| Phase | Scope | Effort | Cumulative value |
|---|---|---|---|
| **Minimum viable** | Deploy PR-Agent, write `AGENT.md`, one custom prompt | 1 week | ~30% |
| **Effective** | \+ context engine, grounding checker, output schema, noise control | 3 weeks | ~70% |
| **Production** | \+ risk routing, policy, security hardening, mutation gates, telemetry | 6–8 weeks | ~90% |
| **Mature** | \+ multi-vendor, attestation, property tests, prompt regression suite | 3–4 months | ~100% |

**Ongoing:** roughly 0.25 FTE to keep it tuned once built. Budget it explicitly.
A review pipeline nobody owns becomes a review pipeline nobody trusts, and then a
review pipeline nobody reads — which costs you money to catch nothing.
