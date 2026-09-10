# The AI Code Review Tooling Landscape

> **Verification notice.** This market changes monthly. Pricing, ownership, licences
> and star counts move constantly — Graphite was acquired by Cursor in December 2025,
> Cursor's Bugbot moved from per-seat to usage-based billing in June 2026, and SonarQube
> Community Edition was renamed Community Build. Everything below was compiled in
> **September 2026**. Re-verify against the vendor's own pricing page before you quote
> a number to your finance team.

---

## Part 1 — How the market is layered

Three generations of tool. They are **layers, not alternatives**, and teams that
treat generation 3 as a replacement for generation 1 trade reproducibility for
eloquence.

| Gen | Class | Mechanism | Strengths | Blind spots |
|---|---|---|---|---|
| **1** | Deterministic analysis | Rules, AST, dataflow, taint tracking | Reproducible, auditable, no data egress, no false-confidence, free | Cannot reason about intent or requirements |
| **2** | Diff-scoped LLM review | Model reads changed lines | Fast, cheap, easy to install | Cannot see anything outside the hunk; misses cross-file and cross-service effects |
| **3** | Agentic / full-context review | Indexes the whole repository, models how the change ripples outward | Catches cross-cutting breakage, convention drift, requirements gaps | Cost, latency, false positives, data egress, non-determinism |

**Layer 1 is not optional.** It is the floor under everything else, it produces the
evidence auditors accept, and it is the only part of the stack that gives the same
answer twice.

---

## Part 2 — Open source tools

### 2.1 LLM-based review tools

| Tool | Repo | Licence | Notes |
|---|---|---|---|
| **Qodo PR-Agent** (formerly CodiumAI) | `github.com/qodo-ai/pr-agent` | MIT from v0.40.0 | The most capable open-source AI reviewer. Self-hostable with your own LLM keys, so code never leaves your provider relationship. Commands for review, describe, improve, ask, test. ~15k stars. Runs as a GitHub Action, GitLab webhook, Bitbucket app or CLI. LLM API cost bills to your own account on every review. |
| **ChatGPT-CodeReview** | `github.com/anc95/ChatGPT-CodeReview` | ISC | The best-maintained lightweight GitHub Action in this class (~4.4k stars, shipped Feb 2026). Diff-scoped. Set up in under an hour. Good first step; will not see beyond the hunk. |
| **ai-codereviewer** | `github.com/villesau/ai-codereviewer` | MIT | The most familiar name in the lightweight tier (~1k stars, high fork count), but **no release since December 2023**. Testing reports roughly a third of suggestions irrelevant. Use as a reference implementation, not a production dependency. |
| **shippie** | `github.com/mattzcarey/shippie` | MIT | The only tool in the lightweight group that reads past the diff — runs an agent with file and shell access. Development paused for eight months before a June 2026 rewrite. Interesting architecture, young. |
| **Kodus** | `github.com/kodustech/kodus-ai` | AGPL-3.0 (+ commercial key for some features) | The most substantial project in this tier — agent-based architecture, very active release cadence. Enterprise controls sit behind a commercial licence key, and documentation on polyglot monorepos is thin. Worth watching. |
| **CodeRabbit CLI / OSS components** | `github.com/coderabbitai` | mixed | Some components open; the reviewer itself is commercial. |
| **Danger** | `github.com/danger/danger-js` · `danger/danger` | MIT | Not AI, but the standard way to express *policy* on PRs (size limits, required labels, changelog rules). Excellent glue for Gate 0. |
| **reviewdog** | `github.com/reviewdog/reviewdog` | MIT | Turns any linter's output into PR review comments. The plumbing layer under a lot of homegrown pipelines. |

### 2.2 Deterministic analysis — the layer you must not skip

| Tool | Repo | Licence | Role |
|---|---|---|---|
| **Semgrep** | `github.com/semgrep/semgrep` · rules: `github.com/semgrep/semgrep-rules` | LGPL-2.1 (engine) | The single most important open-source tool for AI-generated code. Pattern rules that look like the code they match, so you can encode your *own* conventions in an afternoon. This is how you mechanise "cargo-culted pattern" detection. |
| **CodeQL** | `github.com/github/codeql` | MIT (queries); engine free for public repos | Semantic dataflow and taint analysis. Deepest security analysis available in the open. Private repos need GitHub Code Security. |
| **SonarQube Community Build** | `github.com/SonarSource/sonarqube` | LGPL-3.0 | The most mature open-source code-quality platform. Predictable, low-noise, ~21 languages, proven at enterprise scale. Advanced security and reporting are paid. |
| **Ruff** | `github.com/astral-sh/ruff` | MIT | Extremely fast Python linter/formatter. |
| **typescript-eslint** | `github.com/typescript-eslint/typescript-eslint` | MIT | Type-aware JS/TS rules. |
| **Bandit** | `github.com/PyCQA/bandit` | Apache-2.0 | Python security linter. |
| **golangci-lint** | `github.com/golangci/golangci-lint` | GPL-3.0 | Go meta-linter. |
| **PMD / Checkstyle** | `github.com/pmd/pmd` · `github.com/checkstyle/checkstyle` | BSD / LGPL | JVM static analysis. |
| **Infer** | `github.com/facebook/infer` | MIT | Interprocedural analysis for null-deref, leaks, races. |

### 2.3 Supply chain and secrets — failure mode 7

| Tool | Repo | Role |
|---|---|---|
| **OSV-Scanner** | `github.com/google/osv-scanner` | Lockfile vulnerability scanning against the OSV database |
| **Trivy** | `github.com/aquasecurity/trivy` | Vulnerabilities, misconfig, secrets, SBOM — very broad |
| **Grype** / **Syft** | `github.com/anchore/grype` · `github.com/anchore/syft` | Vulnerability scanning and SBOM generation |
| **Gitleaks** | `github.com/gitleaks/gitleaks` | Secret detection, pre-commit and CI |
| **TruffleHog** | `github.com/trufflesecurity/trufflehog` | Secret detection with live credential verification |
| **OpenSSF Scorecard** | `github.com/ossf/scorecard` | Health scoring of dependencies — directly answers "is this new package the model added actually maintained?" |
| **Dependency-Track** | `github.com/DependencyTrack/dependency-track` | Continuous SBOM analysis |

### 2.4 Test efficacy — failure mode 2

This is the category most teams have never installed, and it is the one that
permanently kills tautological tests.

| Tool | Repo | Language |
|---|---|---|
| **mutmut** | `github.com/boxed/mutmut` | Python |
| **Cosmic Ray** | `github.com/sixty-north/cosmic-ray` | Python |
| **Stryker Mutator** | `github.com/stryker-mutator/stryker-js` | JS/TS, .NET, Scala |
| **PIT (pitest)** | `github.com/hcoles/pitest` | JVM |
| **go-mutesting** | `github.com/zimmski/go-mutesting` | Go |
| **cargo-mutants** | `github.com/sourcefrog/cargo-mutants` | Rust |

### 2.5 Stronger oracles — the determinism investment

The techniques Colin Breck argues have become newly affordable. These are what
let an agent verify its own work instead of shipping something for a human to catch.

| Category | Tool | Repo |
|---|---|---|
| Property testing | **Hypothesis** | `github.com/HypothesisWorks/hypothesis` |
| | **fast-check** | `github.com/dubzzz/fast-check` |
| | **jqwik** | `github.com/jqwik-team/jqwik` |
| | **proptest** | `github.com/proptest-rs/proptest` |
| Fuzzing | **AFL++** | `github.com/AFLplusplus/AFLplusplus` |
| | **libFuzzer / OSS-Fuzz** | `github.com/google/oss-fuzz` |
| | **Atheris** (Python) | `github.com/google/atheris` |
| | **cargo-fuzz** | `github.com/rust-fuzz/cargo-fuzz` |
| Model checking | **TLA+ / TLC** | `github.com/tlaplus/tlaplus` |
| | **Alloy** | `github.com/AlloyTools/org.alloytools.alloy` |
| | **Apalache** | `github.com/apalache-mc/apalache` |
| Formal verification | **Kani** (Rust) | `github.com/model-checking/kani` |
| | **CBMC** (C/C++) | `github.com/diffblue/cbmc` |
| | **CrossHair** (Python) | `github.com/pschanely/CrossHair` |
| | **Dafny** | `github.com/dafny-lang/dafny` |
| Deterministic simulation | **TigerBeetle VOPR** | `github.com/tigerbeetle/tigerbeetle` |
| | **madsim** (Rust) | `github.com/madsim-rs/madsim` |
| | **FoundationDB** simulation | `github.com/apple/foundationdb` |
| Undefined behaviour | **Miri** (Rust) · **sanitizers** | `github.com/rust-lang/miri` |

### 2.6 Provenance, attestation and benchmarking

| Tool | Repo | Role |
|---|---|---|
| **Sigstore / cosign** | `github.com/sigstore/cosign` | Keyless signing of attestations — Gate 5 |
| **in-toto** | `github.com/in-toto/in-toto` | Supply-chain attestation format |
| **SLSA** | `github.com/slsa-framework/slsa` | Build provenance levels |
| **Martian Code Review Bench** | `github.com/withmartian/code-review-benchmark` | The first genuinely independent benchmark for AI review agents (Feb 2026), from researchers not selling review tools. Tested a large tool set across ~300k real PRs, measuring which comments developers **actually acted on**. Use its methodology for your own bake-off. |
| **SWE-bench** | `github.com/SWE-bench/SWE-bench` | Agent capability benchmark — relevant context for generation-side trust tiering |

### 2.7 Agent framework building blocks

If you build your own reviewer (as in `02-framework-and-implementation.md`):

| Tool | Repo |
|---|---|
| **Model Context Protocol** | `github.com/modelcontextprotocol` |
| **LiteLLM** — multi-vendor routing, cost tracking, fallback | `github.com/BerriAI/litellm` |
| **Instructor** — structured output enforcement | `github.com/567-labs/instructor` |
| **Outlines** — constrained generation | `github.com/dottxt-ai/outlines` |
| **ts-morph** — TypeScript AST for the grounding checker | `github.com/dsherret/ts-morph` |
| **tree-sitter** — polyglot parsing | `github.com/tree-sitter/tree-sitter` |
| **Promptfoo** — evaluate and regression-test your review prompts | `github.com/promptfoo/promptfoo` |

> That last one is underrated. Your review prompts are production code with no
> tests. Promptfoo lets you build a corpus of PRs with known defects and assert
> that a prompt change did not reduce catch rate.

---

## Part 3 — Commercial tools

### 3.1 Dedicated AI code reviewers

| Tool | Link | Positioning | Deployment | Indicative pricing (verify) |
|---|---|---|---|---|
| **CodeRabbit** | [coderabbit.ai](https://www.coderabbit.ai) | Broadest platform coverage (GitHub, GitLab, Bitbucket, Azure DevOps). Noted for low comment noise. Topped Martian's independent 2026 benchmark on acted-on comments. | SaaS; self-hosted at Enterprise (large seat minimum) | Free for public repos; ~$24/dev/mo Pro annual, ~$48 Pro Plus. Pro has a daily review cap. |
| **Greptile** | [greptile.com](https://www.greptile.com) | Whole-codebase indexing; highest reported recall. Trade-off is a heavier false-positive load. | SaaS; self-host on Enterprise | Free for one active dev; ~$30/seat/mo including a review-credit allowance, overage per review |
| **Qodo Merge** | [qodo.ai](https://www.qodo.ai) | Rules-and-governance oriented: ticket-compliance validation, engineering-standards enforcement, generates tests for bugs it finds. Open-source roots (PR-Agent). | SaaS, single-tenant, **on-premises / air-gapped** | Free tier; ~$19–30/seat/mo; enterprise custom |
| **Cursor Bugbot** | [cursor.com](https://cursor.com) | IDE-native, tight loop with Cursor's agent. Moved from per-seat to usage-based billing in June 2026. | SaaS only | Usage-based, roughly $1–1.50 per run (previously $40/seat) |
| **Graphite Diamond** | [graphite.dev](https://graphite.dev) | AI review bundled into a stacked-diff workflow. Acquired by Cursor, Dec 2025. Limited standalone value if you are not doing stacked PRs. | SaaS only | ~$40/seat/mo Team |
| **Sourcery** | [sourcery.ai](https://sourcery.ai) | Cheapest dedicated reviewer; refactoring suggestions plus review | SaaS | ~$12/dev/mo |
| **CodeAnt AI** | [codeant.ai](https://codeant.ai) | Review plus security scanning and code-quality dashboards | SaaS | ~$24/dev/mo |
| **Korbit AI** | [korbit.ai](https://www.korbit.ai) | Review plus developer-productivity analytics | SaaS | Per-seat |
| **Bito** | [bito.ai](https://bito.ai) | AI review agent with static-analysis integration | SaaS | Per-seat |
| **Ellipsis** | [ellipsis.dev](https://www.ellipsis.dev) | Review plus autonomous fix PRs | SaaS | Per-seat |
| **Macroscope** | [macroscope.com](https://macroscope.com) | Precision-first architecture; GitHub only; usage-based | SaaS | Usage-based, low per-review cost |
| **Tenki Code Reviewer** | [tenki.cloud](https://tenki.cloud) | Full-repo indexing for context-aware review | SaaS | Per-seat |
| **Augment Code** | [augmentcode.com](https://www.augmentcode.com) | Very large-monorepo context and semantic dependency analysis | SaaS | Per-seat |
| **Baz** | [baz.co](https://baz.co) | Change-graph-based review | SaaS | Per-seat |

### 3.2 Platform-native reviewers

| Tool | Link | Notes |
|---|---|---|
| **GitHub Copilot code review** | [github.com/features/copilot](https://github.com/features/copilot) | Zero setup if you already pay for Copilot. Independent testing consistently finds suggestions skew linter-level. Review shares a capped premium-request pool with all other Copilot features. Best as a complement, not a replacement. |
| **GitLab Duo** | [about.gitlab.com/gitlab-duo](https://about.gitlab.com/gitlab-duo/) | Native MR review and summaries for GitLab shops |
| **Amazon Q Developer** | [aws.amazon.com/q/developer](https://aws.amazon.com/q/developer/) | Review, security scanning, AWS-aware; strong fit if you are deep in AWS |
| **Gemini Code Assist** | [cloud.google.com/gemini/docs/codeassist](https://cloud.google.com/gemini/docs/codeassist) | Free tier for GitHub PR review; Google Cloud integration |
| **Claude Code review** | [claude.com/claude-code](https://claude.com/claude-code) | GitHub Action for PR review driven by your `CLAUDE.md`/`AGENT.md`. Token-billed, so cost tracks diff size; the most configurable option if you want the review policy to be *your* files. |

### 3.3 Security and quality platforms (layer 1, commercially supported)

| Tool | Link | Notes |
|---|---|---|
| **Semgrep AppSec Platform** | [semgrep.dev](https://semgrep.dev) | Managed rules, SSO, triage on top of the open engine. ~$30/contributor/mo per product. |
| **Snyk Code** | [snyk.io](https://snyk.io) | SAST + SCA + container + IaC, strong developer workflow |
| **SonarQube Server / Cloud** | [sonarsource.com](https://www.sonarsource.com) | Quality gates, compliance reporting, AI Code Assurance features |
| **Checkmarx One** | [checkmarx.com](https://checkmarx.com) | Enterprise AppSec suite |
| **Veracode** | [veracode.com](https://www.veracode.com) | Long-established, strong compliance reporting |
| **Endor Labs** | [endorlabs.com](https://www.endorlabs.com) | Reachability-based SCA — cuts dependency-alert noise substantially |
| **Codacy** | [codacy.com](https://www.codacy.com) | Quality + security, ~$18/dev/mo |
| **DeepSource** | [deepsource.com](https://deepsource.com) | Static analysis with autofix |
| **Diffblue Cover** | [diffblue.com](https://www.diffblue.com) | Reinforcement-learning Java unit-test generation — deterministic, not LLM |
| **Antithesis** | [antithesis.com](https://antithesis.com) | Deterministic simulation testing on a purpose-built hypervisor. The commercial route to a level-5 oracle. |

---

## Part 4 — Reading benchmarks
For most of this category's life, every published benchmark was won by whoever
published it. Greptile's benchmark had Greptile winning at 82% bug catch. CodeRabbit's
had CodeRabbit winning. Qodo's had Qodo winning. All three are probably honest and
all three are unusable for a purchasing decision.

