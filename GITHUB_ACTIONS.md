# GitHub Actions Integration

## Quick Start

Create `.github/workflows/secureci.yml` in your repository:

```yaml
name: SecureCI Security Scan

on:
  pull_request:
    branches: [main, master, develop]
  push:
    branches: [main, master]

permissions:
  contents: read
  pull-requests: write
  security-events: write
  checks: write

jobs:
  security-scan:
    runs-on: ubuntu-latest
    name: Security Scan
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Run SecureCI
        uses: your-org/secureci@v1
        with:
          fail-on: 'high'
          output-format: 'sarif'
          post-comment: 'true'
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

## Configuration Options

### Basic Configuration

```yaml
- name: Run SecureCI
  uses: your-org/secureci@v1
  with:
    # Path to scan (default: current directory)
    path: './src'
    
    # Fail on severity level (default: high)
    fail-on: 'high'
    
    # Output format: text, json, sarif (default: text)
    output-format: 'sarif'
    
    # Post results as PR comment (default: true)
    post-comment: 'true'
    
    # GitHub token for posting comments
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Module Configuration

```yaml
- name: Run SecureCI
  uses: your-org/secureci@v1
  with:
    # Enable/disable specific modules
    enable-sast: 'true'        # Semgrep SAST scanning
    enable-deps: 'true'        # Dependency scanning
    enable-ai: 'false'         # AI analysis (requires API key)
    enable-checklist: 'true'   # OWASP checklist
```

### AI Analysis

To enable AI analysis with Claude:

```yaml
- name: Run SecureCI with AI
  uses: your-org/secureci@v1
  with:
    enable-ai: 'true'
    anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
```

**Setup:**
1. Get API key from https://console.anthropic.com/
2. Add to repository secrets: Settings → Secrets → Actions → New secret
3. Name: `ANTHROPIC_API_KEY`

## Advanced Workflows

### Scan on PR + Upload SARIF

```yaml
name: Security Scan

on:
  pull_request:

permissions:
  contents: read
  pull-requests: write
  security-events: write

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run SecureCI
        uses: your-org/secureci@v1
        with:
          output-format: 'sarif'
          fail-on: 'high'
      
      # SARIF results automatically uploaded to GitHub Security tab
```

### Multiple Severity Levels

```yaml
jobs:
  critical-scan:
    name: Block on Critical
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: your-org/secureci@v1
        with:
          fail-on: 'critical'
  
  high-scan:
    name: Warn on High
    runs-on: ubuntu-latest
    continue-on-error: true
    steps:
      - uses: actions/checkout@v4
      - uses: your-org/secureci@v1
        with:
          fail-on: 'high'
```

### Scheduled Scans

```yaml
name: Weekly Security Audit

on:
  schedule:
    - cron: '0 0 * * 0'  # Every Sunday at midnight
  workflow_dispatch:      # Manual trigger

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Full Security Scan
        uses: your-org/secureci@v1
        with:
          enable-ai: 'true'
          anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
          fail-on: 'medium'
      
      - name: Upload results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: security-scan-results
          path: secureci-results.*
```

### Matrix Strategy (Multiple Languages)

```yaml
jobs:
  scan:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        path: ['./backend', './frontend', './api']
    steps:
      - uses: actions/checkout@v4
      
      - name: Scan ${{ matrix.path }}
        uses: your-org/secureci@v1
        with:
          path: ${{ matrix.path }}
          fail-on: 'high'
```

### Custom Rules

```yaml
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Create custom config
        run: |
          cat > .secureci.yml << EOF
          fail_on_severity: high
          modules:
            sast: true
            deps: true
            ai: false
            checklist: true
          semgrep_rules:
            - p/owasp-top-ten
            - p/nodejs-scan
            - p/security-audit
          custom_rules_path: ./security-rules
          EOF
      
      - name: Run SecureCI
        uses: your-org/secureci@v1
```

## Output Examples

### PR Comment

When `post-comment: true`, SecureCI posts a formatted comment:

```
## ❌ SecureCI Security Scan FAILED

### 📊 Summary

| Severity | Count |
|----------|-------|
| 🔴 Critical | 2 |
| 🟠 High | 5 |
| 🟡 Medium | 3 |
| 🔵 Low | 1 |
| ℹ️ Info | 0 |

### 🔍 Top Findings

#### 1. 🔴 Hardcoded API Key
**Severity:** CRITICAL
**Module:** checklist
**Location:** `src/auth.ts:42`

Hardcoded API key detected in source code...

**💡 Fix:** Use process.env.API_KEY or equivalent
```

### SARIF Upload

Results appear in GitHub Security tab:
- **Code scanning alerts** - Filterable by severity
- **Security overview** - Trends over time
- **Pull request checks** - Inline annotations

## Troubleshooting

### Scan fails with "Semgrep not found"

The action automatically installs Semgrep. If it fails:

```yaml
- name: Install Semgrep manually
  run: pip install semgrep

- name: Run SecureCI
  uses: your-org/secureci@v1
```

### PR comments not posting

Check permissions:

```yaml
permissions:
  pull-requests: write  # Required for comments
```

### SARIF upload fails

Check permissions:

```yaml
permissions:
  security-events: write  # Required for SARIF
```

### AI module not working

Verify API key is set:

```yaml
- name: Check API key
  run: |
    if [ -z "${{ secrets.ANTHROPIC_API_KEY }}" ]; then
      echo "ANTHROPIC_API_KEY not set"
      exit 1
    fi
```

## Best Practices

### 1. Use Branch Protection

```yaml
# .github/workflows/secureci.yml
name: Security Gate

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: your-org/secureci@v1
        with:
          fail-on: 'high'
```

Then in Settings → Branches → Add rule:
- Require status checks: `Security Gate / security`

### 2. Separate Critical from Warnings

```yaml
jobs:
  critical:
    name: Block Critical
    steps:
      - uses: your-org/secureci@v1
        with:
          fail-on: 'critical'
  
  advisory:
    name: Advisory Scan
    continue-on-error: true
    steps:
      - uses: your-org/secureci@v1
        with:
          fail-on: 'medium'
```

### 3. Cache Dependencies

```yaml
- name: Cache Semgrep
  uses: actions/cache@v3
  with:
    path: ~/.cache/semgrep
    key: semgrep-${{ runner.os }}

- uses: your-org/secureci@v1
```

### 4. Notify on Failures

```yaml
- name: Run SecureCI
  id: scan
  uses: your-org/secureci@v1
  continue-on-error: true

- name: Notify Slack
  if: steps.scan.outcome == 'failure'
  uses: slackapi/slack-github-action@v1
  with:
    payload: |
      {
        "text": "Security scan failed on ${{ github.repository }}"
      }
```

## Exit Codes

- `0` - Scan passed, no issues above threshold
- `1` - Scan failed, issues found at or above threshold

## Environment Variables

SecureCI reads these from GitHub Actions automatically:

- `GITHUB_TOKEN` - For posting comments
- `GITHUB_REPOSITORY` - Owner/repo
- `GITHUB_SHA` - Commit SHA
- `GITHUB_EVENT_PATH` - Event payload

## Local Testing

Test the action locally with `act`:

```bash
# Install act
brew install act

# Run workflow
act pull_request -s GITHUB_TOKEN=your_token
```

## Migration from Other Tools

### From Snyk

```yaml
# Before
- uses: snyk/actions/node@master

# After
- uses: your-org/secureci@v1
  with:
    enable-deps: 'true'
    fail-on: 'high'
```

### From CodeQL

```yaml
# Use both together
- uses: github/codeql-action/init@v3
- uses: your-org/secureci@v1
- uses: github/codeql-action/analyze@v3
```

## Support

- Documentation: https://github.com/your-org/secureci
- Issues: https://github.com/your-org/secureci/issues
- Discussions: https://github.com/your-org/secureci/discussions
