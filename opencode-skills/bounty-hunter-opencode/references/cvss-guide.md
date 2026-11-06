# CVSS 3.1 Scoring Guide

Use this guide to compute the CVSS 3.1 vector string and base score for each finding.

## Vector Format

`CVSS:3.1/AV:{}/AC:{}/PR:{}/UI:{}/S:{}/C:{}/I:{}/A:{}`

---

## Metric Definitions

### Attack Vector (AV)
| Value | Code | When to use |
|-------|------|-------------|
| Network | N | Exploitable remotely over the internet / blockchain |
| Adjacent | A | Requires access to the same network segment |
| Local | L | Requires local access to the machine |
| Physical | P | Requires physical access |

**Default for most web/smart contract findings: N**

### Attack Complexity (AC)
| Value | Code | When to use |
|-------|------|-------------|
| Low | L | No special conditions; attack succeeds reliably |
| High | H | Requires specific configuration, race condition, or difficult-to-control state |

**If the attack needs a race window, specific token type, or flash loan: consider H**

### Privileges Required (PR)
| Value | Code | When to use |
|-------|------|-------------|
| None | N | No auth required |
| Low | L | Normal user account |
| High | H | Admin / privileged role required |

### User Interaction (UI)
| Value | Code | When to use |
|-------|------|-------------|
| None | N | No victim interaction needed |
| Required | R | Victim must click a link, approve a tx, etc. |

### Scope (S)
| Value | Code | When to use |
|-------|------|-------------|
| Unchanged | U | Impact stays within the vulnerable component |
| Changed | C | Impact extends beyond (e.g., compromised server affects other users, bridge affects other chains) |

**Cross-chain attacks, XSS, and SSRF to internal networks often qualify for C**

### Confidentiality (C)
| Value | Code | Description |
|-------|------|-------------|
| None | N | No confidentiality impact |
| Low | L | Limited info disclosure |
| High | H | Full disclosure of sensitive data |

### Integrity (I)
| Value | Code | Description |
|-------|------|-------------|
| None | N | No integrity impact |
| Low | L | Some data can be modified |
| High | H | Complete loss of integrity / funds drained |

### Availability (A)
| Value | Code | Description |
|-------|------|-------------|
| None | N | No availability impact |
| Low | L | Reduced performance, partial DoS |
| High | H | Complete DoS, contract bricked |

---

## Common Patterns

| Finding Type | Typical Vector | Score Range |
|---|---|---|
| Remote Code Execution (RCE) | `AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H` | 10.0 |
| SQL Injection (data exfiltration) | `AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N` | 9.1 |
| SSRF (internal cloud metadata) | `AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:N/A:N` | 8.6 |
| IDOR (financial / account takeover) | `AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:N` | 8.1 |
| Stored XSS (admin context) | `AV:N/AC:L/PR:L/UI:R/S:C/C:H/I:H/A:N` | 8.7 |
| Broken Authentication / JWT bypass | `AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N` | 9.1 |
| IDOR (sensitive read only) | `AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:N/A:N` | 6.5 |
| Web Race Condition (double-spend) | `AV:N/AC:H/PR:L/UI:N/S:U/C:N/I:H/A:N` | 6.3 |
| Subdomain Takeover | `AV:N/AC:L/PR:N/UI:N/S:C/C:L/I:L/A:N` | 6.1 |
| GraphQL Introspection (production) | `AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N` | 5.3 |
| Reflected XSS | `AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N` | 6.1 |
| CSRF (state changing) | `AV:N/AC:L/PR:N/UI:R/S:U/C:N/I:H/A:N` | 6.5 |

---

## Scoring Instructions

1. Select each metric value from the definitions above.
2. Construct the vector string: `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N`
3. Compute the base score using the CVSS 3.1 formula (or reference the common patterns table for well-known attack types).
4. In the report, show: vector string, numerical score, and a one-line justification for each metric selection.

**Example:**
```
CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H → 10.0
AV:N — exploitable from the internet via any RPC node
AC:L — no race condition or special state required
PR:N — no authentication required, any address can call
UI:N — no victim interaction
S:C — compromise of the bridge affects all destination chains
C:H / I:H / A:H — attacker can drain all bridged assets and brick the contract
```
