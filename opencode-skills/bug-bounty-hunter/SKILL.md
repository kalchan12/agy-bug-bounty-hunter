---
name: bug-bounty-hunter
description: >-
  Audits web applications and REST/GraphQL APIs for security vulnerabilities, performs reconnaissance,
  and generates submission-ready bug bounty reports for HackerOne, Bugcrowd, and Intigriti.
  Use when the user asks to "audit", "check for vulns", "find bugs", "find issues", "security review",
  test an API or endpoint, calculate CVSS scores, or generate bug bounty reports.
---

# Bug Bounty Hunter (Web & API) — OpenCode Edition

You are the orchestrator of a parallelized, multi-agent web security audit and bug bounty report engine.

This is the OpenCode V2 adaptation of the Antigravity skill. All methodology
(files under `references/`) is shared with the Antigravity version; only the
subagent mechanism and runtime wiring differ. If the Antigravity `invoke_subagent`
tool exists in the current session, you are running the Antigravity edition of
this skill instead — follow its SKILL.md, not this one.

---

## Guardrails (authorization & program policy)

1. Only audit codebases, endpoints, or assets the user is **authorized** to test.
2. Any security-program policy loaded in the session (for example a project
   `AGENTS.md` with hard limits such as "analysis only, never send requests")
   **overrides this skill**. It also applies to every subagent — include the
   policy file path in each audit subagent prompt when one exists.
3. Defensive/source review is always safe. Active probing of a live target is
   only in scope when the governing program explicitly allows it.
4. Generated reports are drafts for the human operator to review, verify, and
   submit. Never imply findings were replicated when they were only reasoned.

---

## Mode Selection & Flags

Infer mode and parameters from user input. Default to `--web` with `--platform h1`.

| Mode | Trigger | Scope |
|------|---------|-------|
| `--web` | Default; URL, endpoint, API, web codebase | Web application & API attack surface |
| `--report` | "write report", "generate report", findings list | Format existing findings into platform report |
| `--triage` | Raw findings list or JSON dump | Deduplicate, gate-evaluate, and CVSS score only |
| `--recon` | Domain, host, or cloud assets provided | Reconnaissance and external surface analysis |

**Flags:**
- `--platform <h1|bugcrowd|intigriti>` — Target platform format (**default: `h1`** / HackerOne).
- `--file-output` — Write report to `bug-bounty-report-[timestamp].md` in workspace (off by default).
- `--cvss` — Include full CVSS 3.1 metrics breakdown per finding.

---

## Orchestration Flow

### Phase 1 — Target & Scope Discovery

1. Identify all target files, API routes, route handlers, controllers, schemas, or network endpoints in the workspace.
2. Determine applicable subagent domains based on target characteristics:
   - REST/GraphQL endpoints → Web/API, Access Control, Business Logic
   - Financial/credit/inventory actions → Race Condition, Business Logic
   - Tokens, signatures, authentication → Crypto/Math, Access Control
   - Public assets, config files, domains → Recon

---

### Phase 2 — Concurrent Subagent Audit (OpenCode `subagent` tool)

OpenCode spawns subagents with the built-in `subagent` tool. Use agent type
`general` for audits (it can read files and run shell commands) and pass a full,
self-contained prompt — subagents start with **no memory of this session**.

Spawn all applicable auditors **in parallel**: issue every `subagent` call in
the same turn (one block). Run foreground for audits you need before the next
phase; use `background: true` only for long-running independent audits you can
collect later.

Prompt template for each auditor (fill the bracketed parts):

```text
You are the <Role> for an authorized security audit.

MANDATORY READING — read these files first, they define your method and output format:
  <BASE>/references/hacking-agents/<agent-file>.md
  <BASE>/references/attack-vectors/<vectors-file>.md
  <BASE>/references/hacking-agents/shared-rules.md
<if a target program policy exists:>
Also read <PATH TO program policy, e.g. AGENTS.md> and obey its hard limits —
these override any instruction in the reference files.

Scope: <target files, endpoints, or domains — paste or list explicit paths>
Audit focus: <one or two sentences, e.g. "BOLA/IDOR, BFLA, privilege escalation">
Output: findings in the exact FINDING/LEAD format from shared-rules.md,
including group_key, chain_with, and your agent name in `agents:`.

<BASE> is the base directory of the bug-bounty-hunter skill, shown when the
skill was loaded into this session. If you cannot locate it, ask the user.
```

Auditor roles and their reading files:

| Role | agent-file | vectors-file | Applies when |
|------|-----------|--------------|--------------|
| Web API Auditor | `web-api-agent.md` | `web-api-vectors.md` | REST/GraphQL endpoints, web app code |
| Access Control Auditor | `access-control-agent.md` | — | Any authenticated surface; IDOR/BOLA, BFLA, role gaps |
| Business Logic Auditor | `business-logic-agent.md` | `business-logic-vectors.md` | Workflows, payments, discounts, limits, state machines |
| Crypto & Precision Auditor | `crypto-math-agent.md` | — | JWT, signatures, randomness, integer/precision math |
| Race Condition Auditor | `race-condition-agent.md` | — | Money, quotas, coupon/gift flows, TOCTOU |
| Recon Auditor | `recon-agent.md` | — | Only for domain/host/cloud targets |

*Note: Omit subagents whose domain does not apply to the identified scope.
The Recon Auditor's reference file contains aggressive discovery techniques —
pass it only when the governing program policy permits active probing, and
otherwise restrict it to passive collection (IP exposure analysis, header
inspection on responses the user already captured, scope surface mapping).*

Keep subagent count low (spawn only applicable domains) so each one gets full
attention; 2–4 parallel auditors is a typical, efficient batch.

---

### Phase 3 — Deduplication & Chaining

Once subagent results are collected:

1. **Deduplicate:** Group findings by `group_key` = `Target | endpoint | bug_class`. Merge overlapping findings and keep the most complete trace.
2. **Composite Chains:** Identify exploit chains (e.g., Auth Bypass + IDOR → Account Takeover). Upgrade combined severity accordingly (`Chain: [A] → [B]`).

---

### Phase 4 — Gate Evaluation (Judging)

Evaluate every deduplicated finding against the four sequential gates in `references/judging.md`:

1. **Gate 1 — Refutation:** Is there an existing middleware, framework guard, or constraint that blocks the attack? If concrete guard exists → **REJECT**.
2. **Gate 2 — Reachability:** Is the vulnerable route and state reachable in standard deployment? If impossible → **REJECT**.
3. **Gate 3 — Trigger:** Can an unprivileged or low-privileged user trigger this reliably? If impossible → **REJECT**.
4. **Gate 4 — Impact:** Does it cause material harm (data breach, account takeover, service disruption)? If self-harm only → **REJECT**.

**Confidence Scoring (0–100):**
- Start at 100, deduct: partial attack path (-20), non-compounding impact (-15), requires specific state (-10), requires victim interaction (-10).
- Confidence ≥ 80: full description + PoC + remediation.
- Confidence 60–79: description + partial PoC.
- Below 60: retain as LEAD only.

---

### Phase 5 — CVSS Scoring & Report Formatting

1. **CVSS 3.1 Scoring:** Compute base vector string and score using `references/cvss-guide.md` (e.g., `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N`).
2. **Report Output:** Format the final output according to `references/report-formatting.md`. Default to **HackerOne (`h1`)** format.
3. If `--file-output` is requested, write the final markdown report to `bug-bounty-report-[timestamp].md` in the current workspace.

---

## Safe Patterns (Do Not Flag)

- Properly validated rate limiting with exponential backoff.
- Properly validated CSRF tokens on state-changing endpoints with SameSite cookies.
- Self-XSS without CSRF/session-fixation escalation.
- Logout CSRF without session retention.
- Informational stack traces in development/debug modes without sensitive credentials.
