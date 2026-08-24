# Quatrain GitHub Actions

This repository centralizes reusable, composite GitHub Actions for the Quatrain organization's CI/CD workflows. By mutualizing standard build-time operations and versioning tasks, we eliminate duplicate scripts and ease monorepo maintenance.

---

## 🧭 Shared Composite Actions

### 1. `prepare-package-json`
This action prepares `package.json` configurations prior to container packing by replacing workspace version placeholders (`workspace:*` and `*`) with their latest real, published version from the NPM registry.

* **Path**: `./prepare-package-json`
* **Inputs**:
  * `package_json_path`: Path to the target `package.json` file to modify (default: `package.json`).
  * `dependency_prefix`: The prefix of package dependency names that should have their `*` versions resolved from the NPM registry (default: `@quatrain/`).

#### Usage Example
```yaml
- name: Prepare package.json for container build
  uses: Quatrain/actions/prepare-package-json@v1
  with:
    package_json_path: 'containers/api-gateway/package.json'
    dependency_prefix: '@quatrain/'
```

---

### 2. `push-tag`
This action automates the standard package release versioning. It bumps the version using `npm version`, configures Git credentials, commits the `package.json` update, tags the release, and pushes both the commit and release tag back to the origin repository.

* **Path**: `./push-tag`
* **Inputs**:
  * `bump_type`: The type of semantic version bump (`patch`, `minor`, `major`) (default: `patch`).

#### Usage Example
```yaml
- name: Bump patch version & push tag
  uses: Quatrain/actions/push-tag@v1
  with:
    bump_type: 'patch'
```

---

### 3. `publish-package-preview`
Automates on-demand pre-release preview package publishing (with version suffix `-prXX` and dist-tag `prXX`) to NPM and GitHub Packages, complete with automated sticky PR comments for QA testing.

* **Path**: `./publish-package-preview`
* **Inputs**:
  * `pr_number`: Pull Request number (default: `${{ github.event.pull_request.number }}`).
  * `npm_token`: NPM registry authentication token.
  * `github_token`: GitHub token (for registry and PR comment updates).
  * `script_path`: Path to publish script (default: `bin/publish_all.js`).
  * `tag_prefix`: Dist-tag prefix (default: `pr`).

#### Usage Example (in Monorepo CI workflow)
```yaml
- name: Publish On-Demand QA Preview Packages
  if: github.event_name == 'pull_request' && contains(github.event.pull_request.labels.*.name, 'qa:preview')
  uses: Quatrain/actions/publish-package-preview@main
  with:
    npm_token: ${{ secrets.NPM_TOKEN }}
    github_token: ${{ secrets.GITHUB_TOKEN }}
```

---

---

### 4. `deploy-app-preview`
Builds and pushes an ephemeral container image tagged `:prXX` (e.g. `ghcr.io/quatrain/studio:pr25`), triggers optional ArgoCD sync, and notifies the PR with the live staging preview URL (`https://studio-pr25.apps.quatrain.dev`) for QA testing.

* **Path**: `./deploy-app-preview`
* **Inputs**:
  * `pr_number`: Pull Request number (default: `${{ github.event.pull_request.number }}`).
  * `app_name`: Application identifier (e.g. `studio`, `api-gateway`, `studio-web`).
  * `containerfile`: Path to `Containerfile` (default: `Containerfile`).
  * `context`: Build context path (default: `.`).
  * `base_domain`: Staging base domain (default: `dev.brad.team`, e.g. `apps.quatrain.dev`).
  * `github_token`: GitHub Token.
  * `argocd_server`: ArgoCD Server URL (optional).
  * `argocd_auth_token`: ArgoCD Auth Token (optional).

---

### 5. `cleanup-app-preview`
Automatically deletes and tears down the ephemeral ArgoCD preview application and associated Kubernetes resources when a PR is merged or closed without merge.

* **Path**: `./cleanup-app-preview`
* **Inputs**:
  * `pr_number`: Pull Request number (default: `${{ github.event.pull_request.number }}`).
  * `app_name`: Application identifier (e.g. `studio`, `api-gateway`).
  * `base_domain`: Staging base domain (e.g. `apps.quatrain.dev`).
  * `github_token`: GitHub Token.
  * `argocd_server`: ArgoCD Server URL (optional).
  * `argocd_auth_token`: ArgoCD Auth Token (optional).

---

### 🌟 Complete On-Demand QA Preview Lifecycle Workflow (`.github/workflows/preview.yml`)

Here is how to set up the complete on-demand staging preview lifecycle in any application repository (e.g. `CoreApps`):

```yaml
name: Staging Preview Lifecycle

on:
  pull_request:
    types: [opened, synchronize, reopened, labeled, closed]

jobs:
  deploy-preview:
    name: Deploy On-Demand App Preview
    if: github.event.action != 'closed' && contains(github.event.pull_request.labels.*.name, 'qa:preview')
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      pull-requests: write
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Deploy App Preview to Staging
        uses: Quatrain/actions/deploy-app-preview@main
        with:
          pr_number: ${{ github.event.pull_request.number }}
          app_name: 'studio'
          base_domain: 'apps.quatrain.dev'
          github_token: ${{ secrets.GITHUB_TOKEN }}
          argocd_server: ${{ secrets.ARGOCD_SERVER }}
          argocd_auth_token: ${{ secrets.ARGOCD_TOKEN }}

  cleanup-preview:
    name: Cleanup Staging Preview
    if: github.event.action == 'closed'
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
    steps:
      - name: Teardown App Preview from Staging
        uses: Quatrain/actions/cleanup-app-preview@main
        with:
          pr_number: ${{ github.event.pull_request.number }}
          app_name: 'studio'
          base_domain: 'apps.quatrain.dev'
          github_token: ${{ secrets.GITHUB_TOKEN }}
          argocd_server: ${{ secrets.ARGOCD_SERVER }}
          argocd_auth_token: ${{ secrets.ARGOCD_TOKEN }}
```

---

## ⚖️ License
Licensed under the **GNU Affero General Public License v3.0 (AGPL v3)**. See [LICENSE.md](./LICENSE.md) for details.
