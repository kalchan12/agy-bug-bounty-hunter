# Finding Gate Evaluation (Judging)

Every deduplicated finding passes four sequential gates. Fail any gate → **REJECTED** or **DEMOTED** to LEAD. Later gates are not evaluated for failed findings.

---

## Gate 1 — Refutation

Construct the strongest argument that the finding is wrong. Find the guard, check, or constraint that kills the attack — quote the exact line and trace how it blocks the claimed step.

- **Concrete refutation** (a specific guard blocks the exact claimed step) → **REJECTED** (or **DEMOTE** if a code smell remains worth investigating)
- **Speculative refutation** ("probably wouldn't happen", "likely intended") → **clears**, continue to Gate 2

---

## Gate 2 — Reachability

Prove the vulnerable state exists in a live/deployed system.

- **Structurally impossible** (enforced invariant in code prevents the state from ever occurring) → **REJECTED**
- **Requires privileged actions outside normal operation** (owner must misconfigure, multi-sig must collude) → **DEMOTE**
- **Achievable through normal usage, standard workflows, common user actions** → **clears**, continue to Gate 3

---

## Gate 3 — Trigger

Prove an unprivileged (or minimally privileged) actor can execute the attack.

- **Only a trusted role can trigger** → **DEMOTE** (report as medium/low, not critical)
- **Requires impossible preconditions** (victim must perform irrational sequence of actions) → **REJECTED**
- **Unprivileged actor can trigger reliably** → **clears**, continue to Gate 4

---

## Gate 4 — Impact

Prove material harm to an identifiable victim or system.

- **Self-harm only** (attacker affects only their own account, no other victim) → **REJECTED**
- **Low-impact, non-compounding, informational** → **DEMOTE** to low/informational
- **Material loss to identifiable victim** (account takeover, unauthorized data access, service disruption, data breach) → **CONFIRMED**

---

## Severity Adjustment After Gates

After all four gates clear, adjust severity downward if:
- Attack requires specific timing / tight race window: **-1 severity level**
- Attack requires extensive user interaction: **-1 severity level**
- Impact is bounded (limited scope of leaked info, no privilege escalation): **-1 severity level**
- Fix is already deployed in production but not in reviewed branch: **DEMOTE** + note

---

## LEAD Promotion Before Finalization

Before finalizing the LEAD list, promote where warranted:

1. **Multi-agent convergence:** 2+ agents flagged the same endpoint/feature, finding was demoted (not rejected) → promote to FINDING at confidence 75.
2. **Partial path completion:** Only weakness is an incomplete trace, but path is reachable and unguarded → promote to FINDING at confidence 75, no fix required.
3. **Web cross-feature:** Confirmed auth bypass in endpoint A enables IDOR in endpoint B → chain them and promote.

---

## Do Not Report

- Linter warnings, compiler suggestions, gas micro-optimizations.
- Missing NatSpec or event emissions.
- Centralization risk without an exploit path.
- Admin privileges functioning as designed (unless the design itself is the vulnerability).
- Implausible preconditions that require the protocol to already be compromised.
- Self-XSS without any escalation path.
- Logout CSRF without session fixation.
- Rate limiting that genuinely prevents exploitation in the defined threat model.
