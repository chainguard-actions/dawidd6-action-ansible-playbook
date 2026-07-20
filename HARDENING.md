<!-- markdownlint-disable -->

# Hardening Report: dawidd6--action-ansible-playbook/v5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **dawidd6--action-ansible-playbook/v5** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### hardcoded-credentials (severity: high)

A literal OpenSSH private key is hardcoded directly in the workflow's `env:` block under `SSH_PRIVATE_KEY`. The full private key material (-----BEGIN OPENSSH PRIVATE KEY----- ... -----END OPENSSH PRIVATE KEY-----) is embedded in plaintext in the repository. This exposes the private key to anyone with read access to the repository and should be replaced with a GitHub Actions secret reference (e.g., `${{ secrets.SSH_PRIVATE_KEY }}`).

Locations:

- `.github/workflows/test.yml:14`

### unpinned-uses (severity: high)

Two `uses:` references in .github/workflows/test.yml pin to a mutable version tag (`actions/checkout@v5`) rather than an immutable 40-character commit SHA. If the tag is moved (e.g., by a compromised upstream), the workflow will silently execute different code. Both occurrences should be pinned to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v5`.

Locations:

- `.github/workflows/test.yml:27`
- `.github/workflows/test.yml:55`

### missing-permissions (severity: medium)

The workflow file .github/workflows/test.yml has no top-level `permissions:` key, and neither the `remote` job nor the `local` job defines a job-level `permissions:` block. This means the workflow runs with GitHub's default permissions (which include `contents: write` for workflows triggered on push/pull_request in many configurations), granting broader access than necessary. A minimal `permissions:` block (e.g., `contents: read`) should be added at the top level or per job.

Locations:

- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** hardcoded-credentials, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings in .github/workflows/test.yml: (1) Replaced the hardcoded OpenSSH private key in the env block with `${{ secrets.SSH_PRIVATE_KEY }}` secret reference. (2) Pinned both `actions/checkout@v5` references to the full immutable SHA `93cb6efe18208431cddfb8368fd83d5badbf9bfd` with `# v5` comment for readability. (3) Added a top-level `permissions: contents: read` block to enforce least-privilege access.

