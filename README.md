```
███████╗███████╗ ██████╗██╗   ██╗██████╗ ███████╗ ██████╗██╗    
██╔════╝██╔════╝██╔════╝██║   ██║██╔══██╗██╔════╝██╔════╝██║    
███████╗█████╗  ██║     ██║   ██║██████╔╝█████╗  ██║     ██║    
╚════██║██╔══╝  ██║     ██║   ██║██╔══██╗██╔══╝  ██║     ██║    
███████║███████╗╚██████╗╚██████╔╝██║  ██║███████╗╚██████╗██║    
╚══════╝╚══════╝ ╚═════╝ ╚═════╝ ╚═╝  ╚═╝╚══════╝ ╚═════╝╚═╝
```

**Automated security scanner for Pull Requests**

We started building SecureCI from personal pain: working on a team without a Security Engineer, and every time a PR is merged, no one checks the code for vulnerabilities.

SecureCI runs 4 scanners in parallel: SAST (Semgrep), dependency check (CVE), AI analysis (Claude), OWASP checklist. In 60 seconds — report with findings.

**No Security Engineer needed. No $25K/year for Snyk.**

Related project: [ChakLoad-CLI](https://github.com/Aisrefot-Reed/ChakLoad-CLI) - load testing tool.

---

## 🚀 Quick Start

### GitHub Actions (recommended)

```yaml
# .github/workflows/secureci.yml
name: Security Scan
on: [pull_request]

permissions:
  contents: read
  pull-requests: write
  security-events: write

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: your-org/secureci@v1
        with:
          fail-on: 'high'
```

[Full documentation →](GITHUB_ACTIONS.md)

### CLI (local)

```bash
npm install -g secureci

# Interactive menu
secure-ci

# Direct scan
secure-ci scan ./src --fail-on high

# SARIF for GitHub Security
secure-ci scan --sarif
```

## 📋 Commands

```bash
secure-ci              # Interactive menu (local)
secure-ci scan         # Scan (CI/CD)
secure-ci scan --json  # JSON output
secure-ci scan --sarif # SARIF for GitHub Security
secure-ci scan --pr    # Post PR comment
secure-ci install      # Install Semgrep/OSV
secure-ci init         # Create .secureci.yml
```

## 📊 Exit Codes

- `0` - Clean, no issues
- `1` - Issues found

CI systems use this to determine status.

## 📁 Structure

```
secureci/
├── src/
│   ├── index.ts              # CLI entry point
│   ├── modules/              # SAST, AI, Checklist
│   └── rules/                # Security rules (YAML)
├── tests/
│   ├── fixtures/             # Test data
│   └── e2e/                  # E2E tests
└── apps/dashboard/           # Next.js dashboard
```

## 🔒 Detected Vulnerabilities

- Hardcoded passwords, API keys, JWT secrets
- SQL injection patterns
- eval() usage
- Insecure HTTP endpoints
- Sensitive data in logs
- .env files in commits

## 🤝 Feedback

If you're a developer or DevOps in a small team — let us know, we'd love to talk about your experience with security in CI/CD.

**#devsecops #secureci #devtools #opensource #chakbild**

## 📄 License

MIT
