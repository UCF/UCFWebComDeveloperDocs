## Overview

[UCF Packagist](https://github.com/UCF/ucf-packagist) is a private Composer package registry for UCF's WordPress themes and plugins. It is built using [Satis](https://github.com/composer/satis), a static Composer repository generator. The generated registry is hosted via GitHub Pages at [https://ucf.github.io/ucf-packagist/](https://ucf.github.io/ucf-packagist/).

The repository tracks a list of UCF GitHub repositories in `satis.json`. When packages are added or release new versions, GitHub Actions workflows rebuild the static registry output (stored in the `docs/` directory) and commit the updated files back to the repository.

---

## Repository Structure

| Path | Description |
|---|---|
| `satis.json` | Defines the registry name, homepage, and the list of all tracked package repositories |
| `docs/` | The generated static Composer registry, served via GitHub Pages |
| `.github/workflows/` | GitHub Actions workflows for building and updating the registry |

---

## Adding a New Package

When a new UCF repository needs to be tracked by the Packagist registry, use the **Add Package** workflow. This automates adding the repository to `satis.json` and rebuilding the registry.

### Steps

1. Navigate to the [ucf-packagist Actions tab](https://github.com/UCF/ucf-packagist/actions/workflows/satis-add-package.yml).
2. Click **Run workflow**.
3. Fill in the two required inputs:
   - **Package URL** — The Git URL of the repository to add (e.g. `git@github.com:UCF/My-New-Plugin`).
   - **Package Name** — The Composer package name in `vendor/package` format (e.g. `ucf/my-new-plugin`).
4. Click **Run workflow** to start the job.

### What the Workflow Does

The `satis-add-package.yml` workflow uses the [`UCF/satis-add-package`](https://github.com/UCF/satis-add-package) GitHub Action, which:

1. Checks out the `ucf-packagist` repository.
2. Adds the new repository entry to `satis.json`.
3. Runs Satis to rebuild the full registry into `docs/`.
4. Commits the updated `satis.json` and `docs/` files back to `main` (committed as `mcatech`).

> **Note:** After running this workflow, you must also add the `trigger-packagist-update.yml` workflow to the new repository (see [Adding the Update Trigger to a Repository](#adding-the-update-trigger-to-a-repository)).

---

## How Tracked Repositories Trigger Updates

Each tracked repository contains a workflow file at `.github/workflows/trigger-packagist-update.yml`. This workflow fires automatically when a new release is **published** on that repository, and it triggers a partial rebuild of the Packagist registry.

### Example Workflow (`trigger-packagist-update.yml`)

```yaml
name: Trigger Packagist Rebuild

on:
  release:
    types: [published]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - env:
          GIT_EMAIL: mcatech@ucf.edu
          GIT_NAME: mcatech
        run: |
          curl -XPOST \
            -H "Authorization: Bearer ${{ secrets.PACKAGIST_REPO_PAT }}" \
            -H "Accept: application/vnd.github.everest-preview+json" \
            -H "Content-Type: application/json" \
            https://api.github.com/repos/UCF/ucf-packagist/actions/workflows/partial-composer-build.yml/dispatches \
            --data '{"ref": "main", "inputs": {"package": "ucf/my-package-name"}}'
```

The `curl` call uses the GitHub REST API to trigger a `workflow_dispatch` event on the `partial-composer-build.yml` workflow in `ucf-packagist`, passing the Composer package name as the `package` input. Authentication is provided by the `PACKAGIST_REPO_PAT` repository secret.

### Adding the Update Trigger to a Repository

When adding a new repository to the registry, this workflow file must be added to that repository as well:

1. Create `.github/workflows/trigger-packagist-update.yml` in the tracked repository.
2. Use the template above, replacing the `package` value in the `--data` JSON with the repository's Composer package name (e.g. `ucf/my-new-plugin`).
3. Ensure the `PACKAGIST_REPO_PAT` secret is present in the repository (see [Managing the Personal Access Token](#managing-the-personal-access-token)).

---

## Manually Rebuilding the Registry

### Partial Rebuild

A partial rebuild updates a single package's data in the registry. This is faster than a full rebuild and is what the automated release trigger calls.

1. Navigate to the [Partial Packagist Rebuild workflow](https://github.com/UCF/ucf-packagist/actions/workflows/partial-composer-build.yml).
2. Click **Run workflow**.
3. Enter the Composer package name to rebuild (e.g. `ucf/ucf-alert-plugin`).
4. Click **Run workflow**.

### Full Rebuild

A full rebuild regenerates the entire registry from scratch. Use this after adding packages, after making changes to `satis.json` by hand, or if the registry is suspected to be out of sync.

1. Navigate to the [Full Packagist Rebuild workflow](https://github.com/UCF/ucf-packagist/actions/workflows/full-composer-build.yml).
2. Click **Run workflow**, then confirm.

Both rebuild workflows commit the updated `docs/` output back to `main` as `mcatech`.

---

## Managing the Personal Access Token

The `PACKAGIST_REPO_PAT` secret is a GitHub Personal Access Token (PAT) belonging to the **mcatech** account. It is stored as a repository secret on every tracked repository and is used by the `trigger-packagist-update.yml` workflow to authenticate the API call that dispatches the partial rebuild on `ucf-packagist`.

### Required PAT Permissions

- Read access to metadata
- Read and write access to workflows and actions

### Updating the PAT

When the PAT is rotated or expires, it must be updated across all tracked repositories. The [`UCF/github-secrets-management`](https://github.com/UCF/github-secrets-management) utility automates this by pushing the new secret value to every repository in the UCF organization at once.

#### Prerequisites

1. Clone the `github-secrets-management` repository and install its dependencies:

   ```bash
   git clone https://github.com/UCF/github-secrets-management.git
   cd github-secrets-management
   npm install
   ```

2. Have a GitHub token available with sufficient permissions to read repository public keys and write repository secrets across the UCF organization.

3. Generate the new PAT from the **mcatech** account with the required permissions listed above.

#### Running the Update

```bash
node index.js \
  --token <YOUR_ADMIN_GITHUB_TOKEN> \
  --secret PACKAGIST_REPO_PAT \
  --value <NEW_PAT_VALUE> \
  --org UCF
```

Or using short flags:

```bash
node index.js -t <YOUR_ADMIN_GITHUB_TOKEN> -s PACKAGIST_REPO_PAT -v <NEW_PAT_VALUE> -o UCF
```

This will update the `PACKAGIST_REPO_PAT` secret on every repository in the UCF GitHub organization.

> **Note:** The `--token` flag here is your own admin token used to authenticate the secret-management script — not the new PAT value itself. The new PAT value is passed via `--value`.

#### After Updating

Verify the update worked by publishing a test release on one of the tracked repositories and confirming that the `trigger-packagist-update.yml` workflow runs successfully and the partial build completes in `ucf-packagist`.
