<!-- markdownlint-disable -->

# Hardening Report: dawidd6--action-ansible-playbook/v6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **dawidd6--action-ansible-playbook/v6** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### hardcoded-credentials (severity: high)

A literal OpenSSH private key is hardcoded in the `SSH_PRIVATE_KEY` environment variable in the `remote` job of test.yml. The full private key material (-----BEGIN OPENSSH PRIVATE KEY----- ... -----END OPENSSH PRIVATE KEY-----) is embedded directly in the workflow YAML rather than being stored as a GitHub Actions secret. This exposes the private key to anyone with read access to the repository.

Locations:

- `.github/workflows/test.yml:14`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags or branch names instead of full 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks if the referenced tag or branch is moved or compromised. Failing references: `actions/checkout@v6` (appears twice in test.yml) and `dawidd6/reusable-workflows/.github/workflows/npm-updates.yml@master` (in npm-updates.yml).

Locations:

- `.github/workflows/test.yml:36`
- `.github/workflows/test.yml:47`
- `.github/workflows/npm-updates.yml:6`

### github-env-injection (severity: high)

In the 'Setup remote' step of test.yml, the output of `ssh-keyscan localhost` (an external command whose output is not controlled by the workflow) is written directly to `$GITHUB_ENV` via `echo $(ssh-keyscan localhost 2>/dev/null) >> $GITHUB_ENV` without first sanitizing newlines with `printf '%s' ... | tr -d '\n\r'`. A multi-line value written to GITHUB_ENV without sanitization can inject additional environment variable assignments, potentially allowing environment variable hijacking.

Locations:

- `.github/workflows/test.yml:41`

### permissions (severity: medium)

Neither workflow file has a top-level `permissions:` key, and no individual job within them defines a `permissions:` block. Without explicit permissions, workflows run with the default (potentially broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/test.yml:1`
- `.github/workflows/npm-updates.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** hardcoded-credentials, unpinned-uses, github-env-injection, permissions

**Notes:**

Fixed all four findings in .github/workflows/test.yml and .github/workflows/npm-updates.yml:
1. hardcoded-credentials: Replaced the literal OpenSSH private key with `${{ secrets.SSH_PRIVATE_KEY }}` — key material removed from source.
2. unpinned-uses: Pinned actions/checkout@v6 → @d23441a48e516b6c34aea4fa41551a30e30af803 (both occurrences) and dawidd6/reusable-workflows@master → @eef24d408f08a926601a42fd4051807bcf3d3569.
3. github-env-injection: Replaced `echo $(ssh-keyscan localhost 2>/dev/null) >> $GITHUB_ENV` with `ssh-keyscan localhost 2>/dev/null | tr -d '\r' | grep -v '^$' >> $GITHUB_ENV` to strip newlines/carriage-returns that could inject extra env var assignments.
4. permissions: Added `permissions: {}` at the workflow level in both files to enforce least privilege.

