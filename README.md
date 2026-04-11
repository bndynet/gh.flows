# gh.flows

This repository provides reusable GitHub Actions workflows.

---

## 1) `.github/workflows/npm.yml`

### Example usage


```yaml
name: Build+Publish

on:
  release:
    types: [published]
  workflow_dispatch:
    inputs:
      publish_enabled:
        description: 'Whether to publish to npm'
        required: false
        default: false
        type: boolean
      publish-version:
        description: 'Version bump type'
        required: false
        default: none
        type: choice
        options:
          - none
          - major
          - minor
          - patch

permissions:
  contents: write
  id-token: write

jobs:
  publish:
    uses: bndynet/gh.flows/.github/workflows/npm.yml@main
    with:
      publish-enabled: ${{ github.event_name == 'release' || inputs.publish_enabled }} 
      publish-version: ${{ inputs['publish-version'] }}
      publish-auth-method: 'token'                                # default: token (or oidc)                         
      deploy-folder: ''
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}                          # when publish-enabled=true and publish-auth-method=token
```

```yaml
name: Build+Publish

on:
  release:
    types: [published]
  workflow_dispatch:
    inputs:
      publish_enabled:
        description: 'Whether to publish to npm'
        required: false
        default: false
        type: boolean
      publish-version:
        description: 'Version bump type'
        required: false
        default: none
        type: choice
        options:
          - none
          - major
          - minor
          - patch

permissions:
  contents: write
  id-token: write

jobs:
  publish:
    uses: bndynet/gh.flows/.github/workflows/npm.yml@main
    with:
      node-version: '24'                                           # default: '24'
      working-directory: '.'                                       # default: '.'
      registry-url: 'https://registry.npmjs.org'                   # default registry
      lint-command: 'npm run lint'                                 # default: 'npm run lint'
      test-command: 'npm test'                                     # default: 'npm test'
      build-command: 'npm run build'                               # default: 'npm run build'
      publish-enabled: ${{ github.event_name == 'release' || inputs.publish_enabled }} 
      publish-version: ${{ inputs['publish-version'] }}            # default: none, patch/minor/major
      publish-auth-method: 'token'                                 # default: token (or oidc)
      publish-tag: 'latest'                                        # default: 'latest'
      publish-provenance: true                                     # default: true
      deploy-folder: ''                                            # default: '', e.g., 'dist,build,.next'
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}                          # when publish-enabled=true and publish-auth-method=token
```

