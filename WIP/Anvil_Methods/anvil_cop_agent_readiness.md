# Agent Readiness CoP preparation

A framework for measuring and improving how well your codebase supports autonomous development. Evaluate repositories across eight technical pillars and five maturity levels.

## /readiness-report
 Run /readiness-report to see where you stand across eight technical pillars and five maturity levels, with specific recommendations for what to fix first.
 
 **Invisible Bottleneck in AI Coding Tasks (Applies to Subsequent Tasks)**

* Uneven performance of AI coding agents is commonly misattributed to the model or agent choice.
* The primary limiting factor is typically the **codebase environment**, not the agent.

**Key Environment Failures**

* Absence of pre-commit hooks → slow feedback (e.g. long CI wait times instead of immediate validation).
* Undocumented environment variables → repeated agent trial-and-error and failures.
* Build or test processes dependent on undocumented tribal knowledge (e.g. Slack threads) → agent cannot self-verify changes.

**Principles**

* These are **environment and process deficiencies**, not AI capability issues.
* Poor feedback loops and unclear instructions compound inefficiency and negate agent effectiveness.
* Fast feedback, explicit documentation, and deterministic build processes significantly amplify the effectiveness of any AI coding agent.

### CLI: /readiness-report
Run /readiness-report in Droid to evaluate any repository. The report shows your current level, which criteria pass and fail, and prioritized suggestions for what to fix first.

## Technical Pillars: Anvil / Python Apps (Anvil.works–Specific)

### 1. Style & Validation

Automated checks to catch errors immediately and enforce consistency across server and client code.

**Anvil/Python Examples**

* `black` for formatting
* `ruff` or `flake8` for linting
* `mypy` (where applicable) for type checking
* Pre-commit hooks running locally

**Without this**

* Agent introduces style or syntax issues, deploys to Anvil, waits for runtime failure, iterates blindly.

---

### 2. Build & Run Model (Anvil Context)

Clear, documented rules for how code is executed in Anvil’s runtime.

**Anvil/Python Examples**

* Documented separation of **client code vs server modules**
* Explicit notes on startup behaviour (module import side effects)
* Clear guidance on background tasks and scheduled jobs
* CI checks that mirror Anvil runtime constraints

**Without this**

* Agent places logic in the wrong execution context or relies on unsupported runtime behaviour.

---

### 3. Testing

Fast, deterministic tests that can run outside Anvil before deployment.

**Anvil/Python Examples**

* `pytest` for server-module logic
* Mocking Anvil APIs (`anvil.server`, `anvil.tables`)
* Test suite runnable locally in < 1 minute
* Test commands documented

**Without this**

* Agent cannot validate changes locally and breaks the deployed app.

---

### 4. Documentation

Explicit capture of Anvil-specific conventions and constraints.

**Anvil/Python Examples**

* `AGENTS.md` describing:

  * Where business logic belongs (server vs client)
  * How to access Data Tables safely
  * Naming and module conventions
* `README.md` covering app structure
* Architecture notes for forms, services, and tables

**Without this**

* Agent guesses Anvil conventions and produces structurally incorrect code.

---

### 5. Development Environment

Reproducible local tooling aligned with Anvil’s Python runtime.

**Anvil/Python Examples**

* Pinned Python version matching Anvil
* `requirements.txt` for local testing tools
* Mock environment variables documented
* Clear guidance on what *cannot* be reproduced locally

**Without this**

* Agent encounters environment mismatches invisible to human developers.

---

### 6. Code Quality

Clear boundaries and small units of logic to fit agent context limits.

**Anvil/Python Examples**

* Server modules organised by responsibility
* Minimal logic in Forms; UI delegates to server functions
* Small, single-purpose functions
* Avoidance of large “god” server modules

**Without this**

* Agent cannot reason about data flow or dependencies within context limits.

---

### 7. Observability

Actionable runtime feedback from deployed Anvil apps.

**Anvil/Python Examples**

* Structured logging in server modules
* Consistent error messages returned to clients
* Centralised error tracking (where available)
* Explicit logging around Data Table access and background tasks

**Without this**

* Agent receives vague runtime failures with no diagnostic signal.

---

### 8. Security & Governance

Controls aligned with Anvil’s secrets and permission model.

**Anvil/Python Examples**

* Clear rules for:

  * Server-only secrets
  * Never exposing secrets to client code
* Documented use of Anvil’s built-in secrets management
* Review requirements for Data Table schema changes

**Without this**

* Agent leaks secrets into client code or weakens access controls.

---

**Principle**
In Anvil apps, agent effectiveness is constrained primarily by **clarity of structure, runtime rules, and feedback**, not model capability. A well-prepared Anvil environment enables reliable, safe, and high-leverage agent contributions.

## 5 Maturity levels
Repositories progress through five levels. Each level represents a qualitative shift in what autonomous agents can accomplish.

### Level 1: Functional
Code runs, requires manual setup
Basic tooling signals that every repository should have. Without these, development is unpredictable for humans and impossible for agents.

Key Signals

README exists
Linter configured
Type checker active
Unit tests present
Agent Capability

Agents struggle with simple tasks. High failure rate, constant intervention.

Examples

Personal projects,
Early prototypes

### Level 2: Documented
Workflows are written down
Documentation and basic automation exist. New contributors can onboard with docs alone.

Key Signals

AGENTS.md exists
Reproducible dev env
Pre-commit hooks
Branch protection
Agent Capability

Simple tasks with supervision. Bug fixes, small features.

Examples

?
?
?

### Level 3: Standardized
Production-ready for agents
Clear processes defined and enforced. Minimum bar for production-grade autonomous operation.

Key Signals

E2E tests exist
Docs maintained
Security scanning
Observability
Agent Capability

Routine maintenance: bug fixes, tests, docs, dependency upgrades.

Examples

?
?
?
Level 3 is the target. Most teams should aim here first.

### Level 4: Optimized
Fast feedback loops
Systems designed for productivity. Sub-minute feedback on code quality.

Key Signals

Sub-minute validation
Full observability
Canary deploys
Build optimization
Agent Capability

Complex multi-step tasks. Features, refactoring, migrations.

Examples

?
?

### Level 5: Autonomous
Self-improving systems
Sophisticated orchestration and self-healing. Few organizations reach this level.

Key Signals

Task decomposition
Multi-service orchestration
Self-healing
Auto-remediation
Agent Capability

Portfolio management. Humans set direction, agents execute.

Examples

Aspirational for most


## Evaluation:
### Consistent Evaluations

* Agent Readiness evaluates 60+ criteria using LLM-based analysis.
* LLM non-determinism can cause score variance between consecutive runs on the same repository.

**Stabilisation Approach**

* Each evaluation is grounded against the repository’s previous readiness report.
* This enforces continuity and reduces scoring drift.

**Observed Impact**

* Pre-grounding variance:

  * Average: 7%
  * Peaks: up to 14.5%
* Post-grounding variance:

  * Average: 0.6%
  * Sustained over six weeks

**Validation Scope**

* Results confirmed across 9 benchmark repositories
* Coverage includes low, medium, and high readiness tiers


### Scoring Model

* All criteria are **binary**: pass or fail.
* Signals are objective and machine-verifiable:

  * File existence (e.g. `AGENTS.md`, `CODEOWNERS`)
  * Configuration presence and validity (linters, tests, branch protection)
  * Ability to execute checks locally

**Evaluation Scope**

* **Repository-scoped criteria**: evaluated once per repository

  * e.g. branch protection enabled, `CODEOWNERS` present
* **Application-scoped criteria**: evaluated per app in monorepos

  * e.g. linter and tests configured for each app
  * Results expressed as ratios (e.g. `3/4 apps pass`)

**Level Progression**

* Each level requires:

  * ≥ 80% of criteria at that level, **and**
  * 100% of criteria from all preceding levels
* Enforces dependency order and foundational maturity before higher-level capabilities

**Organization Metrics**

* Tracked as percentage of active repositories reaching Level 3+
* Example: “80% of active repositories are agent-ready”
* Favoured over aggregate or averaged scores

---

### Automated Remediation

* Readiness reports support **automated fixes** via CLI or dashboard.
* Remediation runs an agent that opens pull requests to address failed criteria.

**Automated Fix Scope**

* Foundational, high-impact gaps:

  * Missing documentation (`AGENTS.md`, `README`)
  * Absent or misconfigured linters
  * Missing pre-commit hooks
  * Baseline tooling setup

**Operational Model**

* Changes are applied via standard pull requests.
* Fixes complete in minutes rather than days of manual effort.
* Readiness checks are re-run post-merge to validate remediation and update scores.

---

### Compounding Effect

* Improvements compound over time:

  * Better environments → more effective agents
  * More effective agents → increased capacity
  * Increased capacity → further environment improvements
* Teams that measure and iterate systematically gain accelerating advantage.
* The maturity gap between teams widens over time.

**Tool-Agnostic Benefit**

* Agent readiness improvements are not platform-specific.
* Any AI-assisted development workflow benefits from the same environmental investments.
* Returns persist regardless of agent or vendor choice.

