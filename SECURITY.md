# Security Model

## Why No External Actions?

GitHub's action restrictions are **recursive** - they check not only the action you call, but also every action used internally.

### The Problem

If your workflow uses:
```yaml
uses: compose-network/github-actions/actions/local-testnet@v0.2.0
```

And that action internally uses:
```yaml
uses: actions/checkout@v4
uses: actions/setup-go@v6
uses: docker/setup-buildx-action@v3
```

GitHub will **block** them if you set:
```
Allowed actions: compose-network/github-actions@*
```

**You cannot wrap external actions to bypass security restrictions** - this is by design.

### The Solution

Replace ALL external actions with native shell commands:

| External Action | Replaced With |
|----------------|---------------|
| `actions/checkout@v4` | Native `git clone` |
| `actions/setup-go@v6` | Direct download from go.dev |
| `docker/setup-buildx-action@v3` | Native `docker buildx` commands |

Now you can set the strictest restrictions and it works!

## Security Benefits

### 1. Hardcoded Repository URLs

Repository URLs are hardcoded in the actions and cannot be overridden:

```yaml
# In actions/local-testnet/action.yml
--publisher-url=https://github.com/compose-network/publisher.git
--op-geth-url=https://github.com/compose-network/op-geth.git
```

Users can only specify branch names:
```yaml
with:
  publisher-branch: 'my-branch'  # ✅ Allowed
  publisher-url: 'https://...'   # ❌ Not possible
```

**Prevents:** Attackers from cloning malicious repositories.

### 2. Zero External Dependencies

All actions use only native commands:
- `bash`, `git`, `curl`, `wget`, `docker`, `jq`

No third-party GitHub Actions are used.

**Prevents:**
- Supply chain attacks through compromised actions
- Automatic updates breaking your workflows
- External action dependencies being removed

### 3. Tagged Versions Only

Configure repositories to only allow tagged versions:

```
Settings → Actions → General
→ Allowed actions: compose-network/github-actions@*
```

Then in runner group:
```
Workflow access: Selected workflows
→ owner/repo/.github/workflows/*.yml@refs/tags/*
```

**Prevents:**
- Workflows running from untested branches
- Automatic updates to actions
- Code injection via branch manipulation

## Configuration

### For Repositories Using These Actions

```
Settings → Actions → General → Actions permissions
→ Allow select actions and reusable workflows
→ Allowed actions: compose-network/github-actions@*
```

This restricts the repository to ONLY use actions from `compose-network/github-actions`.

### For Runner Groups (Restricted Runners)

```
Settings → Actions → Runner groups → public-isolated

Repository access: Selected repositories
  → compose-network/publisher
  → compose-network/op-geth

Workflow access: Selected workflows
  → compose-network/publisher/.github/workflows/*.yml@refs/tags/*
  → compose-network/op-geth/.github/workflows/*.yml@refs/tags/*
```

This ensures:
- Only specific repositories can use the runner
- Only specific workflow files can run
- Only tagged releases (not branches) can trigger workflows

## What This Prevents

With this configuration, malicious actors **cannot**:

❌ Use `actions/checkout` to clone their own malicious repository
❌ Use any external third-party actions
❌ Override repository URLs to point to malicious code
❌ Run workflows from branches (only tags allowed)
❌ Create new unauthorized workflow files
❌ Use the runner from unauthorized repositories

## Attack Scenarios Blocked

### Scenario 1: Malicious Checkout
**Attack:** PR contributor tries to checkout their malicious repo
```yaml
- uses: actions/checkout@v4
  with:
    repository: attacker/malicious-code
```
**Blocked by:** `actions/checkout` not in allowed actions list

### Scenario 2: Custom Repository URLs
**Attack:** Try to override repository URLs
```yaml
- uses: compose-network/github-actions/actions/local-testnet@v0.2.0
  with:
    publisher-url: 'https://github.com/attacker/publisher.git'
```
**Blocked by:** Input doesn't exist - URLs are hardcoded

### Scenario 3: Branch Injection
**Attack:** Modify code in `main` branch to inject malicious code
**Blocked by:** Workflows can only run from `@refs/tags/*`, not branches

### Scenario 4: Compromised External Action
**Attack:** `actions/checkout@v4` gets compromised by attackers
**Blocked by:** We don't use external actions - only native commands

## Trade-offs

### Advantages
✅ Maximum security - zero external dependencies
✅ No supply chain risk
✅ Fully auditable code
✅ Works with strictest restrictions

### Disadvantages
⚠️ More maintenance - must update native commands manually
⚠️ More verbose - more lines of code than using actions
⚠️ Platform-specific - commands assume Linux environment

## Recommended Configuration

**Step 1:** Repository Settings
```
compose-network/github-actions@*
```

**Step 2:** Runner Group
```
Workflows: *.yml@refs/tags/*
```

**Step 3:** Workflow File
```yaml
uses: compose-network/github-actions/actions/local-testnet@v0.2.0
```

This gives you **maximum security** with minimal complexity.
