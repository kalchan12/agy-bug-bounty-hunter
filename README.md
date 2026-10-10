# Antigravity Bug Bounty Hunter (`agy-bug-bounty-hunter`)

> Parallelized multi-agent security audit and bug bounty skill built natively for Google Antigravity (`agy`). Focuses exclusively on Web & API security, automated triage, and submission-ready reports for HackerOne, Bugcrowd, and Intigriti.

---

## Overview

Bug Bounty Hunter runs **specialized security audit subagents in parallel** using Antigravity's native `invoke_subagent` multi-agent engine. Findings are deduplicated, passed through 4 verification gates, scored with CVSS 3.1, and formatted directly for bug bounty platform submission.

| Subagent | Focus Area |
|---|---|
| **Web / API** | Auth bypass, IDOR, SQLi, SSRF, XSS, GraphQL, file upload flaws |
| **Access Control** | Broken Object Level Authorization (BOLA/IDOR), BFLA, vertical/horizontal escalation |
| **Business Logic** | Workflow skipping, payment/promo abuse, rate limit circumvention |
| **Crypto & Precision** | JWT algorithm confusion, timing attacks, weak RNG, numeric/rounding errors |
| **Race Conditions** | HTTP/2 concurrent requests, balance double-spends, voucher races, TOCTOU |
| **Recon** | Exposed configs, leaked secrets, subdomain takeovers, cloud misconfigurations |

**Default Report Format:** HackerOne (`h1`) Markdown with CVSS 3.1 vectors, reproducible steps, and remediation guidance.

---

## Installation

### In Any Workspace Project
Clone directly into your project's `.agents/skills` directory:

```bash
mkdir -p .agents/skills
git clone git@github.com:kalchan12/agy-bug-bounty-hunter.git .agents/skills/bug-bounty-hunter
```

### Global Machine Install (Available Across All Workspaces)
Clone into `~/.agents/skills/` and register in `~/.gemini/config/skills.json`:

```bash
mkdir -p ~/.agents/skills
git clone git@github.com:kalchan12/agy-bug-bounty-hunter.git ~/.agents/skills/bug-bounty-hunter

mkdir -p ~/.gemini/config
cat << 'EOF' > ~/.gemini/config/skills.json
{
  "entries": [
    {
      "path": "~/.agents/skills"
    }
  ]
}
EOF
```

---

## Usage

Trigger naturally inside Antigravity CLI (`agy`) or IDE:

```bash
# Audit an API codebase or route directory
"Audit the API routes in src/api"

# Perform a security review on endpoints
"Check this web application for security vulnerabilities"

# Triage existing notes or generate a report
"Generate a HackerOne report for these findings"
```

### Flags & Options
- `--platform <h1|bugcrowd|intigriti>` — Report format (default: `h1`)
- `--cvss` — Include full CVSS 3.1 metrics breakdown
- `--file-output` — Save report directly to `bug-bounty-report-[timestamp].md`
