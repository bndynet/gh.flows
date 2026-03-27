# gh.flows

This repository provides reusable GitHub Actions workflows.

---

## 1) `.github/workflows/npm.yml`

### Example usage

```yaml
name: Publish Package

on:
  release:
    types: [published]
  workflow_dispatch:

permissions:
  contents: read
  id-token: write

jobs:
  publish:
    uses: bndynet/gh.flows/.github/workflows/npm.yml@main
    with:
      node-version: '20'                           # default: '20'
      working-directory: '.'                       # default: '.'
      registry-url: 'https://registry.npmjs.org'   # default registry
      lint-command: 'npm run lint'                 # default: 'npm run lint'
      test-command: 'npm test'                     # default: 'npm test'
      build-command: 'npm run build'               # default: 'npm run build'
      publish-enabled: false                       # default: false
      publish-auth-method: 'token'                 # default: token (or oidc)
      publish-tag: 'latest'                        # default: 'latest'
      publish-provenance: true                     # default: true
      artifact-paths: ''                           # default: '', e.g., 'dist,build,.next'
      artifact-name: 'artifacts'                   # default: 'artifacts' 
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}          # when publish-enabled=true and publish-auth-method=token
```

---

## 2) `.github/workflows/deploy-gh-pages.yml`

### Example usage

```yaml
name: Deploy Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: write
  id-token: write

jobs:
  deploy:
    uses: bndynet/gh.flows/.github/workflows/deploy-gh-pages.yml@main
    with:
      artifact-name: ''         # default: '', e.g. 'artifacts'
      deploy-folder: 'public'   # default: 'public'
      target-branch: 'gh-pages' # default: 'gh-pages'
    secrets:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```
