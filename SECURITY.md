# Security Configuration Guide

## Overview

This repository provides composite actions designed to work with **restricted GitHub runners** that only allow actions from trusted repositories. This prevents security vulnerabilities where malicious actors could execute unauthorized code.

## Threat Model

### Risks Without Restrictions

Without proper restrictions, a malicious actor with PR access could:

1. **Checkout malicious repositories**: Use `actions/checkout` to clone their own repo with malicious code
2. **Run arbitrary commands**: Execute bash commands or scripts from untrusted sources
3. **Access secrets**: Steal sensitive environment variables and secrets
4. **Modify build artifacts**: Inject backdoors into Docker images or binaries
5. **Escalate privileges**: Exploit runner permissions to access other resources

### How This Repo Mitigates Risks

By using ONLY actions from `compose-network/github-actions`, you ensure:

- ✅ All code is reviewed and trusted
- ✅ No external dependencies that could be compromised
- ✅ Git operations are controlled (checkout only from approved repos)
- ✅ All commands are predefined and auditable
- ✅ No arbitrary script execution from PR contributors
- ✅ **Repository URLs are hardcoded** - Users can only specify branches, not repository URLs
  - OP Geth: `https://github.com/compose-network/op-geth.git`
  - Publisher: `https://github.com/compose-network/publisher.git`
  - Contracts: `https://github.com/compose-network/contracts.git`

## Configuring Restricted Runners

### Option 1: GitHub Enterprise with Action Restrictions

If using GitHub Enterprise, configure allowed actions:

1. Go to your organization/repository Settings
2. Navigate to Actions → General
3. Under "Actions permissions", select "Allow select actions and reusable workflows"
4. Add to the allow list:
   ```
   compose-network/github-actions@*
   ```

### Option 2: Self-Hosted Runner with Network Restrictions

Configure your self-hosted runner to only access specific repositories:

```bash
# Example: Use firewall rules to restrict GitHub API access
# Only allow connections to compose-network/github-actions

iptables -A OUTPUT -d github.com -p tcp --dport 443 -m string \
  --string "compose-network/github-actions" --algo bm -j ACCEPT
iptables -A OUTPUT -d github.com -p tcp --dport 443 -j REJECT
```

### Option 3: Custom Runner with Action Validation

Implement a pre-execution hook that validates actions:

```bash
#!/bin/bash
# validate-action.sh - Runner pre-execution hook

WORKFLOW_FILE=$1

# Extract all 'uses:' lines from workflow
ACTIONS=$(grep -oP "uses:\s*\K[^\s]+" "$WORKFLOW_FILE")

# Check if all actions are from allowed repo
for action in $ACTIONS; do
  if [[ ! $action =~ ^compose-network/github-actions ]]; then
    echo "ERROR: Unauthorized action detected: $action"
    exit 1
  fi
done

echo "All actions validated successfully"
```

## Using Secure Workflows

### Recommended: Secure PR Comment Trigger

Use the `secure-pr-comment-trigger.yml` example which uses ONLY actions from this repo:

```yaml
# ✅ SECURE - All steps use compose-network/github-actions
- uses: compose-network/github-actions/actions/add-reaction@main
- uses: compose-network/github-actions/actions/get-pr-info@main
- uses: compose-network/github-actions/actions/checkout-pr@main
- uses: compose-network/github-actions/actions/local-testnet@main
```

### Avoid: Standard Workflows with External Actions

```yaml
# ❌ INSECURE - Uses external actions that could be compromised
- uses: actions/checkout@v4              # External dependency
- uses: peter-evans/create-or-update-comment@v4  # External dependency
```

## Available Secure Actions

All actions use only native shell commands and have been audited for security:

| Action | Commands Used | Purpose |
|--------|---------------|---------|
| `add-reaction` | `curl` | Add emoji reaction to comments |
| `get-pr-info` | `curl`, `jq` | Fetch PR branch and SHA |
| `checkout-pr` | `git` | Clone and checkout PR code |
| `local-testnet` | `docker`, `curl`, `go`, `git` | Run testnet and tests |

## Security Checklist

Before deploying workflows with restricted runners:

- [ ] Configure runner to only allow `compose-network/github-actions` actions
- [ ] Use `secure-pr-comment-trigger.yml` example as template
- [ ] Review all workflow files to ensure no external actions are used
- [ ] Test workflow in isolated environment first
- [ ] Monitor workflow logs for unauthorized access attempts
- [ ] Regularly update to latest action versions
- [ ] Audit changes to this repository before deploying updates

## Reporting Security Issues

If you discover a security vulnerability in these actions:

1. **DO NOT** create a public issue
2. Email security@compose.network with details
3. Include:
   - Description of the vulnerability
   - Steps to reproduce
   - Potential impact
   - Suggested fix (if available)

## Additional Resources

- [GitHub Actions Security Best Practices](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)
- [Restricting Actions in Enterprise](https://docs.github.com/en/enterprise-cloud@latest/admin/policies/enforcing-policies-for-your-enterprise/enforcing-policies-for-github-actions-in-your-enterprise)
