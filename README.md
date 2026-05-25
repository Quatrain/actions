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
  uses: Quatrain/actions/prepare-package-json@main
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
  uses: Quatrain/actions/push-tag@main
  with:
    bump_type: 'patch'
```

---

## 🔒 License
Proprietary / Closed Source for Quatrain internal infrastructures.
