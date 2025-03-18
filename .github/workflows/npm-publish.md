# Node.js build + publish Workflow

A flexible and secure GitHub Actions workflow for building and publishing Node.js packages to npmjs.

## Features

- 🔐 **Dual Authentication**: Support for both OIDC (passwordless) and traditional token-based authentication
- 📜 **Provenance Support**: Generate cryptographic proof linking your package to the exact build
- 🎯 **Flexible Configuration**: Customizable build commands, working directories, and publish tags
- 🚀 **Security First**: Built on GitHub OIDC for maximum security without storing secrets

## Prerequisites

### For OIDC Authentication (Recommended)

1. **Configure npmjs to trust your repository:**
   - Go to [npmjs.com](https://www.npmjs.com/) → Account Settings → Access
   - Create a "Provenance-based access token"
   - Configure the OIDC subject pattern (default: `repo:bndynet/gh.flows:ref:refs/heads/<branch>`)

2. **GitHub Permissions:**
   The workflow already includes the required permissions:
   ```yaml
   permissions:
     contents: read
     id-token: write  # Required for OIDC
   ```

### For Token Authentication

- Store your npm token as a secret: `NPM_TOKEN`

## Usage

### Basic Example (OIDC + Provenance - Recommended)

```yaml
name: Publish Package

on:
  release:
    types: [published]

jobs:
  publish:
    uses: bndynet/gh.flows/.github/workflows/npm-publish.yml@main
    with:
      node-version: '20'
      publish: true
      auth-method: 'oidc'
      provenance: true
    # No secrets needed!
```

### Advanced Examples

#### 1. **OIDC with Provenance** (Most Secure)

```yaml
jobs:
  publish:
    uses: bndynet/gh.flows/.github/workflows/npm-publish.yml@main
    with:
      node-version: '20'
      working-directory: './packages/my-lib'
      publish: true
      auth-method: 'oidc'
      provenance: true
      publish-tag: latest
      build-command: npm run build
    # No secrets required
```

#### 2. **OIDC without Provenance**

```yaml
jobs:
  publish:
    uses: bndynet/gh.flows/.github/workflows/npm-publish.yml@main
    with:
      node-version: '20'
      publish: true
      auth-method: 'oidc'
      provenance: false
    # Still no secrets needed
```

#### 3. **Token Authentication with Provenance**

```yaml
jobs:
  publish:
    uses: bndynet/gh.flows/.github/workflows/npm-publish.yml@main
    with:
      node-version: '20'
      publish: true
      auth-method: 'token'
      provenance: true
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

#### 4. **Token Authentication Only** (Traditional)

```yaml
jobs:
  publish:
    uses: bndynet/gh.flows/.github/workflows/npm-publish.yml@main
    with:
      node-version: '20'
      publish: true
      auth-method: 'token'
      provenance: false
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

#### 5. **Build Only (No Publish)**

```yaml
jobs:
  build:
    uses: bndynet/gh.flows/.github/workflows/npm-publish.yml@main
    with:
      node-version: '20'
      publish: false
      build-command: npm run build
```

## Input Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `node-version` | string | `'20'` | Node.js version to use |
| `working-directory` | string | `'.'` | Relative path to package directory (where package.json lives) |
| `registry-url` | string | `'https://registry.npmjs.org'` | npm registry URL |
| `auth-method` | string | `'token'` | Authentication method: `'oidc'` or `'token'` |
| `publish` | boolean | `false` | Whether to publish to npm |
| `publish-tag` | string | `'latest'` | npm dist-tag for publish (e.g., `latest`, `next`, `beta`) |
| `build-command` | string | `'npm run build'` | Build command to run (empty to skip) |
| `provenance` | boolean | `true` | Generate provenance attestation (requires npm >= 9.5.0) |

## Secrets

| Secret | Required | Description |
|--------|----------|-------------|
| `NPM_TOKEN` | Only when `auth-method='token'` | npm authentication token |

## Complete Workflow Example

Here's a complete example of how to use this workflow in your repository:

```yaml
# .github/workflows/publish.yml
name: Publish to npm

on:
  release:
    types: [published]
  workflow_dispatch:  # Allow manual triggers

permissions:
  contents: read
  id-token: write

jobs:
  publish:
    uses: your-org/your-repo/.github/workflows/npm-publish.yml@main
    with:
      node-version: '20'
      working-directory: '.'
      publish: true
      auth-method: 'oidc'
      provenance: true
      publish-tag: latest
      build-command: npm run build
```

## Authentication Methods Explained

### OIDC Authentication (Recommended)

**How it works:**
- GitHub provides an OIDC token automatically
- npm verifies the token against your repository
- No secrets stored in GitHub

**Benefits:**
- ✅ No secrets to manage or rotate
- ✅ Automatic token expiration
- ✅ Tied to specific repository/branch
- ✅ More secure than long-lived tokens

**Requirements:**
- npm CLI 9.5.0 or higher
- `id-token: write` permission
- npmjs account configured for OIDC trust

### Token Authentication

**How it works:**
- Uses a long-lived npm token
- Stored as a GitHub secret
- Traditional authentication method

**When to use:**
- Legacy systems
- When OIDC is not available
- Testing environments

## Provenance Explained

**What is Provenance?**
Provenance creates a cryptographic link between your published package and the exact GitHub Actions build that created it. Users can verify:
- Which repository built the package
- Which commit triggered the build
- Which GitHub Actions workflow was used

**Example verification:**
```bash
npm view <package-name>@<version> attestations
```

**Requirements:**
- npm CLI >= 9.5.0
- `id-token: write` permission
- Works with both OIDC and token authentication

## Troubleshooting

### "Missing id-token permission"
Ensure you have:
```yaml
permissions:
  id-token: write
```

### "NPM_TOKEN is required"
You're using `auth-method: 'token'` but didn't provide the secret:
```yaml
secrets:
  NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

### "npm ERR! code EOTP"
Some packages require OTP. Add it to your secrets:
```yaml
secrets:
  NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
  NPM_OTP: ${{ secrets.NPM_OTP }}
```

### "Provenance not showing"
Verify:
1. npm version >= 9.5.0
2. `provenance: true` is set
3. `id-token: write` permission is granted

## Security Best Practices

1. **Use OIDC whenever possible** - No long-lived secrets to manage
2. **Enable Provenance** - Provides supply chain transparency
3. **Limit workflow permissions** - Only grant necessary permissions
4. **Use specific branch references** - Pin to specific commits/tags in production
5. **Review npm audit regularly** - Check for vulnerabilities

## License

This workflow is provided as-is for use in your projects.
