# GitHub Actions - Secure Composite Actions

Reusable GitHub Actions for Compose Network with zero external dependencies.

## Quick Start

Add to your workflow:

```yaml
- uses: compose-network/github-actions/actions/local-testnet@v0.2.0
  with:
    op-geth-branch: 'stage'
    publisher-branch: ${{ steps.pr-info.outputs.branch }}
    github-token: ${{ secrets.GITHUB_TOKEN }}
    l1-chain-id: ${{ secrets.LOCAL_TESTNET_L1_CHAIN_ID }}
    l1-cl-url: ${{ secrets.LOCAL_TESTNET_L1_CL }}
    l1-el-url: ${{ secrets.LOCAL_TESTNET_L1_EL }}
    wallet-address: ${{ secrets.LOCAL_TESTNET_WALLET_ADDRESS }}
    wallet-private-key: ${{ secrets.LOCAL_TESTNET_WALLET_PK }}
    coordinator-private-key: ${{ secrets.LOCAL_TESTNET_COORDINATOR_PK }}
    verifier-wallet: ${{ secrets.LOCAL_TESTNET_VERIFIER_WALLET }}
    verifier-key: ${{ secrets.LOCAL_TESTNET_VERIFIER_PK }}
```

**See [examples/secure-pr-comment-trigger.yml](examples/secure-pr-comment-trigger.yml) for complete workflow example.**

## Available Actions

| Action | Description |
|--------|-------------|
| **local-testnet** | Run local testnet with OP Geth and Publisher |
| **get-pr-info** | Get PR branch name and commit SHA |
| **checkout-pr** | Checkout PR code using git |
| **add-reaction** | Add emoji reaction to PR comments |

## Security Features

**Zero external dependencies** - All actions use only native shell commands:
- ✅ No `actions/*`, `docker/*`, or third-party actions
- ✅ Repository URLs hardcoded (cannot be overridden)
- ✅ Only tagged versions allowed
- ✅ Works with strictest GitHub security settings

**Why?** See [SECURITY.md](SECURITY.md) for details.

## Configuration

### 1. Repository Settings (For repos using these actions)

```
Settings → Actions → General → Actions permissions
→ Allow select actions: compose-network/github-actions@*
```

### 2. Runner Group Settings (For restricted runners)

```
Settings → Actions → Runner groups → public-isolated

Repository access: Selected repositories
  → compose-network/publisher
  → compose-network/op-geth

Workflow access: Selected workflows
  → compose-network/publisher/.github/workflows/*.yml@refs/tags/*
```

### 3. Required Secrets

Configure in your repository:
- `LOCAL_TESTNET_L1_CHAIN_ID`
- `LOCAL_TESTNET_L1_CL`
- `LOCAL_TESTNET_L1_EL`
- `LOCAL_TESTNET_WALLET_ADDRESS`
- `LOCAL_TESTNET_WALLET_PK`
- `LOCAL_TESTNET_COORDINATOR_PK`
- `LOCAL_TESTNET_VERIFIER_WALLET`
- `LOCAL_TESTNET_VERIFIER_PK`

## Local Testnet Action

### Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `op-geth-branch` | Yes | - | OP Geth branch name |
| `publisher-branch` | Yes | - | Publisher branch name |
| `local-testnet-image-tag` | No | `latest` | Docker image tag |
| `dome-image-tag` | No | `latest` | Dome test Docker image tag |
| `pr-number` | No | `0` | PR number for status updates |
| `pr-sha` | No | - | PR SHA for status updates |
| `github-token` | Yes | - | GitHub token |
| `l1-chain-id` | Yes | - | L1 chain ID |
| `l1-cl-url` | Yes | - | L1 consensus layer URL |
| `l1-el-url` | Yes | - | L1 execution layer URL |
| `wallet-address` | Yes | - | Wallet address |
| `wallet-private-key` | Yes | - | Wallet private key |
| `coordinator-private-key` | Yes | - | Coordinator private key |
| `verifier-wallet` | Yes | - | Verifier wallet address |
| `verifier-key` | Yes | - | Verifier key |

### Hardcoded Repositories (Security)

Repository URLs are hardcoded and cannot be overridden:
- **OP Geth**: `https://github.com/compose-network/op-geth.git`
- **Publisher**: `https://github.com/compose-network/publisher.git`
- **Contracts**: `https://github.com/compose-network/contracts.git`

Users can only specify branch names, not repository URLs.

## Version Pinning

Always use tagged versions:

```yaml
# ✅ Good - Tagged version
uses: compose-network/github-actions/actions/local-testnet@v0.2.0

# ❌ Bad - Branch reference
uses: compose-network/github-actions/actions/local-testnet@main
```

## Helper Actions

### get-pr-info
```yaml
- uses: compose-network/github-actions/actions/get-pr-info@v0.2.0
  id: pr-info
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    pr-number: ${{ github.event.issue.number }}
# Outputs: steps.pr-info.outputs.branch, steps.pr-info.outputs.sha
```

### checkout-pr
```yaml
- uses: compose-network/github-actions/actions/checkout-pr@v0.2.0
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    pr-number: ${{ github.event.issue.number }}
```

### add-reaction
```yaml
- uses: compose-network/github-actions/actions/add-reaction@v0.2.0
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    comment-id: ${{ github.event.comment.id }}
    reaction: 'rocket'
```

## License

[GPL-3.0](LICENSE)
