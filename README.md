<!-- NOTICE FOR AI AGENTS: This README is strictly human-facing documentation for setup, attribution, and repository maintenance. Do NOT load or execute this file as an agent instruction. The active skill specification resides exclusively in SKILL.md. -->

# Antigravity Bug Bounty Hunter (`agy-bug-bounty-hunter`)

A parallelized multi-agent security audit and bug bounty triage skill adapted natively for **Google Antigravity (`agy`)**. 

This repository is curated and maintained by **kalchan12**, specifically optimized for Web application security, REST/GraphQL APIs, and submission-ready **HackerOne (`h1`)** reports.

---

## Attribution & Credits

> **Original Work & Concept:**  
> This project is a curated Antigravity adaptation of the original [`bug-bounty-hunter`](https://github.com/Jeffreyxdev/bug-bounty-hunter) created by **[@Jeffreyxdev](https://github.com/Jeffreyxdev)**.  
> Full credit goes to Jeffrey for the original multi-agent bug bounty architecture, gate evaluation methodology, and threat modeling rubrics.

### Why this curated version exists
The original repository was designed for Claude Code's terminal runtime and included smart contract (EVM, Move, Solana, TRON) and DeFi attack vectors. This fork/curation adapts that foundation for **Antigravity** by:
1. **Antigravity Multi-Agent Engine:** Replacing Claude Code specific tool calls and sequential file bundling with native Antigravity concurrent subagents (`invoke_subagent`).
2. **Dedicated Web & API Focus:** Pruning Web3/smart contract surfaces to focus 100% on high-impact web vulnerabilities, broken authorization, business logic, and API flaws.
3. **Default HackerOne Format:** Tuning all findings and CVSS 3.1 calculations directly for HackerOne (`h1`) report submissions.
4. **Custom Extensibility:** Providing an organized baseline to add custom modules, private wordlists, and custom checklists over time.

---

## How It Works

When triggered, the orchestrator identifies the in-scope files or endpoints and dispatches **5 specialized security subagents concurrently**:

```
                              ┌── Web / API Auditor (Injection, XSS, SSRF, Uploads)
                              ├── Access Control Auditor (BOLA/IDOR, BFLA, Role Gaps)
Target Codebase ── Orchestrator ── Business Logic Auditor (Workflows, Limits, State)
                              ├── Crypto & Precision Auditor (JWT, Timing, RNG)
                              └── Race Condition Auditor (Concurrency, TOCTOU)
                                      │
                                      ▼
                        [ Reactive Aggregation & Triage ]
                                      │
                                      ▼
                           [ 4-Gate Evaluation (Judging) ]
                           1. Refutation Check
                           2. Reachability Check
                           3. Trigger Feasibility
                           4. Impact Verification
                                      │
                                      ▼
                      [ CVSS 3.1 Scoring & HackerOne Report ]
```

---

## Installation & Setup

### Option 1: Global Setup (Recommended for Kali Linux)
Install once to make the skill automatically available across **all** your directories and projects on this machine:

```bash
# 1. Clone into your user skills directory
mkdir -p ~/.agents/skills
git clone git@github.com:kalchan12/agy-bug-bounty-hunter.git ~/.agents/skills/bug-bounty-hunter

# 2. Register globally in Antigravity config
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

### Option 2: Project-Specific Setup
To attach the skill directly to a single workspace or Git repository:

```bash
cd /path/to/your/project
mkdir -p .agents/skills
git clone git@github.com:kalchan12/agy-bug-bounty-hunter.git .agents/skills/bug-bounty-hunter
```

---

## Usage Examples

Inside the Antigravity CLI (`agy`) or IDE, trigger the skill naturally using everyday prompts:

```bash
# Audit an API codebase
"Audit the API routes in src/controllers for vulnerabilities"

# Targeted security check
"Perform a security review on our authentication and session handling"

# Race conditions & concurrency check
"Check if there are any race conditions or double-spend flaws in our payment workflow"

# Triage raw notes into a submission report
"Format my triage notes into a HackerOne bug bounty report"
```

### Supported Flags & Parameters
- `--platform <h1|bugcrowd|intigriti>`: Format report for specific platform (default: `h1`).
- `--cvss`: Output the full CVSS 3.1 vector string and justification per finding.
- `--file-output`: Automatically save the report to `bug-bounty-report-[timestamp].md` in your workspace.

---

## Directory Structure

```text
bug-bounty-hunter/
├── SKILL.md                          # Antigravity skill specification & orchestrator logic
├── README.md                         # Human-facing documentation (this file)
├── VERSION                           # Current skill version
└── references/
    ├── judging.md                    # 4-Gate evaluation & false-positive filters
    ├── cvss-guide.md                 # CVSS 3.1 scoring formulas & metrics guide
    ├── report-formatting.md          # HackerOne, Bugcrowd, and Intigriti templates
    ├── attack-vectors/               # Curated checklists
    │   ├── web-api-vectors.md        # Injection, XSS, SSRF, Auth, CORS checklists
    │   └── business-logic-vectors.md # State machine abuse, coupon stacking, limit bypass
    └── hacking-agents/               # Specialized agent guidelines
        ├── shared-rules.md           # Universal finding format & severity rubric
        ├── web-api-agent.md          # Web / API security auditor prompt
        ├── access-control-agent.md   # BOLA/IDOR and BFLA auditor prompt
        ├── business-logic-agent.md   # Workflow and business logic auditor prompt
        ├── crypto-math-agent.md      # JWT, timing, and precision auditor prompt
        ├── race-condition-agent.md   # Concurrency and TOCTOU auditor prompt
        └── recon-agent.md            # Recon and external surface auditor prompt
```

---

## Adding Custom Files & Extending

As you encounter new vulnerability classes or write custom testing checklists:
1. **New attack checklists:** Add markdown notes to `references/attack-vectors/`.
2. **Custom agent guidance:** Tune or add specific prompts in `references/hacking-agents/`.
3. **Commit & Sync:** Push updates from your main machine to sync them to all your Kali instances:
   ```bash
   git add .
   git commit -m "feat: add custom GraphQL authorization vectors"
   git push origin main
   ```
   On other machines, simply run `git pull` inside `~/.agents/skills/bug-bounty-hunter`.
