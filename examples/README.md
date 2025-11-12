# Workflow Examples

This directory contains example workflows demonstrating how to use the local testnet composite action.

## Available Examples

### 1. Secure PR Comment Trigger (`secure-pr-comment-trigger.yml`) ⭐ RECOMMENDED

**This is the most secure option for restricted runners.**

Triggers the local testnet using ONLY actions from this repository when a PR comment contains `/test`.

**Security Features:**
- ✅ Uses only `compose-network/github-actions` actions
- ✅ No external dependencies (no `actions/checkout`, `peter-evans/*`, etc.)
- ✅ Safe for restricted runners that only allow actions from trusted repos
- ✅ Prevents malicious actors from injecting unauthorized code

**Features:**
- Automatically detects PR branch and uses it for the publisher
- Posts test results as PR comments
- Updates commit status
- Requires team member authorization
- Uses `public-isolated` runner (you control the runner)

**How to use:**
1. Copy this file to your repository's `.github/workflows/` directory
2. Ensure all required secrets are configured
3. Configure your runner to only allow actions from `compose-network/github-actions`
4. Comment `/test` on any PR to trigger the workflow

### 2. PR Comment Trigger (`pr-comment-trigger.yml`)

Standard version that uses common third-party actions.

**Features:**
- Uses `actions/checkout@v4` and `peter-evans/create-or-update-comment@v4`
- Simpler workflow structure
- Requires allowing external actions on your runner

**How to use:**
1. Copy this file to your repository's `.github/workflows/` directory
2. Ensure all required secrets are configured
3. Adjust the `runs-on` value if you use a different runner
4. Comment `/test` on any PR to trigger the workflow

### 3. Manual Dispatch (`manual-dispatch.yml`)

Allows manual triggering of the action with custom inputs via the GitHub UI.

**Features:**
- Full control over all workflow parameters
- Can override repository URLs
- Can specify custom branches and image tags
- No PR integration
- Uses `public-isolated` runner (you control the runner)

**How to use:**
1. Copy this file to your repository's `.github/workflows/` directory
2. Ensure all required secrets are configured
3. Adjust the `runs-on` value if you use a different runner
4. Go to Actions → Manual Testnet Run → Run workflow
5. Fill in the desired parameters

## Required Secrets

All examples require these secrets to be configured in your repository:

```
LOCAL_TESTNET_L1_CHAIN_ID
LOCAL_TESTNET_L1_CL
LOCAL_TESTNET_L1_EL
LOCAL_TESTNET_WALLET_ADDRESS
LOCAL_TESTNET_WALLET_PK
LOCAL_TESTNET_COORDINATOR_PK
LOCAL_TESTNET_VERIFIER_WALLET
LOCAL_TESTNET_VERIFIER_PK
```

## Customization

Feel free to modify these examples to fit your needs:

- **Change runners** - Modify `runs-on` to use your preferred runner (e.g., `ubuntu-latest`, `self-hosted`)
- **Change triggers** - Update `on:` to use different events (e.g., `pull_request`, `push`)
- **Change trigger conditions** - Modify the `if:` conditions (e.g., different PR comment keywords)
- **Modify default branch names** - Change which branches are used for op-geth and publisher
- **Add additional steps** - Insert steps before or after the action call
- **Adjust permissions** - Customize the `permissions:` section based on your needs
