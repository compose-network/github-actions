# GitHub Actions - Composite Actions

This repository contains reusable GitHub Actions for the Compose Network project.

## Available Actions

| Action | Description | Location |
|--------|-------------|----------|
| **local-testnet** | Run local testnet with configurable repositories | `actions/local-testnet/` |
| **get-pr-info** | Get PR branch and commit SHA | `actions/get-pr-info/` |
| **checkout-pr** | Checkout PR code using git | `actions/checkout-pr/` |
| **add-reaction** | Add reaction to GitHub comment | `actions/add-reaction/` |

## Security Features

This repository is designed to work with **restricted GitHub runners** that only allow actions from trusted repositories. All actions in this repo use only:

- Native shell commands (`bash`, `curl`, `git`, `jq`)
- Docker commands (for local-testnet)
- No external action dependencies

This prevents potential security vulnerabilities where malicious actors could:
- Checkout unauthorized repositories
- Execute arbitrary code through third-party actions
- Access sensitive secrets through compromised dependencies

**📖 See [SECURITY.md](SECURITY.md) for complete security configuration guide**

**⭐ Use [examples/secure-pr-comment-trigger.yml](examples/secure-pr-comment-trigger.yml) for a fully secure implementation**

## Local Testnet Action

A composite action for running local testnet tests with configurable OP Geth and Publisher branches.

### Location

`actions/local-testnet/action.yml`

### Features

- Configurable OP Geth and Publisher branches
- Hardcoded trusted repository URLs (security feature)
- Configurable Docker image tag for local-testnet
- Automatic PR status updates
- Smoke test execution and result reporting
- Support for PR branch detection

### Repository URLs (Hardcoded)

For security, repository URLs are hardcoded and cannot be overridden:
- **OP Geth**: `https://github.com/compose-network/op-geth.git`
- **Publisher**: `https://github.com/compose-network/publisher.git`
- **Contracts**: `https://github.com/compose-network/contracts.git`

### Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `op-geth-branch` | **Yes** | - | OP Geth branch name to use |
| `publisher-branch` | **Yes** | - | Publisher branch name to use |
| `local-testnet-image-tag` | No | `latest` | Docker image tag for local-testnet |
| `pr-number` | No | `0` | Pull request number (for status updates) |
| `pr-sha` | No | - | Pull request commit SHA (for status updates) |
| `github-token` | **Yes** | - | GitHub token for API access |
| `l1-chain-id` | **Yes** | - | L1 chain ID |
| `l1-cl-url` | **Yes** | - | L1 consensus layer URL |
| `l1-el-url` | **Yes** | - | L1 execution layer URL |
| `wallet-address` | **Yes** | - | Wallet address |
| `wallet-private-key` | **Yes** | - | Wallet private key |
| `coordinator-private-key` | **Yes** | - | Coordinator private key |
| `verifier-wallet` | **Yes** | - | Verifier wallet address |
| `verifier-key` | **Yes** | - | Verifier key |

### Required Secrets

The following secrets must be configured in your repository and passed as inputs:

- `LOCAL_TESTNET_L1_CHAIN_ID`
- `LOCAL_TESTNET_L1_CL`
- `LOCAL_TESTNET_L1_EL`
- `LOCAL_TESTNET_WALLET_ADDRESS`
- `LOCAL_TESTNET_WALLET_PK`
- `LOCAL_TESTNET_COORDINATOR_PK`
- `LOCAL_TESTNET_VERIFIER_WALLET`
- `LOCAL_TESTNET_VERIFIER_PK`

### Outputs

| Output | Description |
|--------|-------------|
| `test-result` | Test result (success or failure) |

### Usage Examples

**Complete example workflows are available in the [`examples/`](examples/) directory:**
- [`examples/pr-comment-trigger.yml`](examples/pr-comment-trigger.yml) - Trigger on PR comments with `/test`
- [`examples/manual-dispatch.yml`](examples/manual-dispatch.yml) - Manual workflow dispatch with inputs

#### Example 1: PR Comment Trigger

This example shows how to trigger the action when a PR comment contains `/test`, using the PR's branch for the publisher:

```yaml
name: Local Testnet on PR Comment

on:
  issue_comment:
    types: [created]

jobs:
  run-testnet:
    runs-on: public-isolated  # You control the runner
    if: |
      github.event.issue.pull_request &&
      contains(github.event.comment.body, '/test') &&
      (github.event.comment.author_association == 'MEMBER' ||
       github.event.comment.author_association == 'OWNER')
    permissions:
      contents: read
      issues: write
      pull-requests: write
      statuses: write
    steps:
      - name: Get PR info
        id: pr-info
        run: |
          PR_DATA=$(curl -s -H "Authorization: token ${{ secrets.GITHUB_TOKEN }}" \
            "https://api.github.com/repos/${{ github.repository }}/pulls/${{ github.event.issue.number }}")
          echo "branch=$(echo "$PR_DATA" | jq -r '.head.ref')" >> "$GITHUB_OUTPUT"
          echo "sha=$(echo "$PR_DATA" | jq -r '.head.sha')" >> "$GITHUB_OUTPUT"

      - name: Checkout PR code
        uses: actions/checkout@v4
        with:
          ref: refs/pull/${{ github.event.issue.number }}/head

      - name: Run local testnet
        uses: compose-network/github-actions/actions/local-testnet@main
        with:
          op-geth-branch: 'stage'
          publisher-branch: ${{ steps.pr-info.outputs.branch }}
          pr-number: ${{ github.event.issue.number }}
          pr-sha: ${{ steps.pr-info.outputs.sha }}
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

#### Example 2: Manual dispatch with custom branches

```yaml
name: Manual Testnet Run

on:
  workflow_dispatch:
    inputs:
      op-geth-branch:
        description: 'OP Geth branch'
        required: true
        default: 'stage'
      publisher-branch:
        description: 'Publisher branch'
        required: true
        default: 'develop'

jobs:
  run-testnet:
    runs-on: public-isolated  # You control the runner
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Run local testnet
        uses: compose-network/github-actions/actions/local-testnet@main
        with:
          op-geth-branch: ${{ inputs.op-geth-branch }}
          publisher-branch: ${{ inputs.publisher-branch }}
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

### How It Works

1. The action accepts configurable inputs for repository URLs and branches
2. Runs the local testnet Docker container with the specified configurations
3. Executes smoke tests from the `dome` repository
4. Posts test results as a comment on the PR (if `pr-number` is provided)
5. Updates commit status (if `pr-sha` is provided)

### Key Differences from Reusable Workflows

This is a **composite action**, not a reusable workflow. Key advantages:

- **You control the runner** - Specify `runs-on` in your workflow (e.g., `public-isolated`)
- **You control triggers** - Define when the workflow runs (`on: issue_comment`, etc.)
- **You control permissions** - Set permissions explicitly in your job
- **More flexible** - Can be used as a step within a larger workflow

### Notes

- Default URLs point to `compose-network` repositories but can be overridden
- The `local-testnet-image-tag` defaults to `latest` but can be set to a specific version
- PR status updates and comments are only posted when `pr-number` and `pr-sha` are provided
- All inputs including secrets must be passed explicitly to the action

## Helper Actions

### Get PR Info (`actions/get-pr-info`)

Retrieves pull request information including branch name and commit SHA.

**Inputs:**
- `github-token` (required): GitHub token
- `pr-number` (required): PR number

**Outputs:**
- `branch`: PR branch name
- `sha`: PR commit SHA

### Checkout PR (`actions/checkout-pr`)

Checks out pull request code using git commands.

**Inputs:**
- `github-token` (required): GitHub token
- `pr-number` (required): PR number
- `repository` (optional): Repository in format owner/repo (defaults to current repo)

### Add Reaction (`actions/add-reaction`)

Adds a reaction emoji to a GitHub comment.

**Inputs:**
- `github-token` (required): GitHub token
- `comment-id` (required): Comment ID
- `reaction` (optional): Reaction type (default: `rocket`)

Valid reactions: `+1`, `-1`, `laugh`, `confused`, `heart`, `hooray`, `rocket`, `eyes`

## Contributing

We welcome community contributions!

- Create a branch, push your changes, and open a PR.

## License

Repository is distributed under [GPL-3.0](LICENSE).
