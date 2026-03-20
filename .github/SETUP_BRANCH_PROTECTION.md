# Enabling Required Build Checks for Pull Requests

This guide explains how to configure GitHub so that the **Build NMSE** workflow
must pass before a pull request can be merged. The workflow file
(`.github/workflows/build-nmse.yml`) already runs on `pull_request` events;
you only need to enable the branch-protection rule described below.

## How the Workflow Operates

| Event | Build & Test | Upload Artifact | Update Release |
|-------|:------------:|:---------------:|:--------------:|
| **push** (to default branch) | ✅ | ✅ (pre-zipped) | ✅ (`latest` tag) |
| **pull_request** | ✅ | ❌ | ❌ |
| **workflow_dispatch** | ✅ | ✅ (pre-zipped) | ✅ (`latest` tag) |

- **PR builds are never exposed to end-users.** They only verify that the
  code compiles and tests pass.
- **Push builds** produce a pre-zipped artifact and update the rolling
  `latest` GitHub Release. The NMSE.Site download button links directly to
  this release asset so users never need to visit the Actions page.

## Prerequisites

- You must be a repository **admin** (or org owner) to change branch-protection rules.
- The `Build NMSE` workflow must have run at least once so GitHub recognizes the
  status check name. The easiest way is to open a test PR that touches any source
  file, or trigger the workflow manually from the **Actions** tab.

## Steps

### 1. Open Branch Protection Settings

1. Go to your repository on GitHub.
2. Click **Settings** → **Branches** (under *Code and automation*).
3. Under **Branch protection rules**, click **Add branch protection rule**
   (or edit an existing rule for your default branch, e.g. `main`).

### 2. Configure the Rule

| Setting | Value |
|---------|-------|
| **Branch name pattern** | `main` (or your default branch name) |
| **Require a pull request before merging** | ✅ Enabled |
| **Require status checks to pass before merging** | ✅ Enabled |
| **Require branches to be up to date before merging** | ✅ Recommended |
| **Status checks that are required** | Search for and add **`build`** (the job name from the workflow) |

> **Tip:** If the `build` check does not appear in the search, trigger the
> workflow manually first (Actions → Build NMSE → Run workflow), then return
> to this page.

### 3. Save

Click **Create** (or **Save changes**) at the bottom of the page.

## Porting to a Different Repository

The workflow and site are designed to be portable:

1. Copy `.github/workflows/build-nmse.yml` into the new repository.
2. Update the `env` variables at the top of the workflow file if project paths differ:
   - `PROJECT_FILE` – path to the main `.csproj`
   - `TEST_PROJECT_FILE` – path to the test `.csproj`
   - `DOTNET_VERSION` – target .NET SDK version
   - `BUILD_CONFIGURATION` – `Release` or `Debug`
3. Adjust the `paths` filters under `push` and `pull_request` to match the
   new repository's folder structure.
4. In `NMSE.Site/js/site.js`, update the `SITE_CONFIG` object:
   - `owner` – GitHub user or organisation
   - `repo` – repository name
   - `releaseTag` – must match the tag used in the workflow (default: `latest`)
5. Follow the **Steps** above to enable branch protection in the new repository.

## Verifying

After setup, any pull request that targets the protected branch will:

1. Trigger the **Build NMSE** workflow automatically.
2. Show a required status check on the PR. Merging is blocked until the
   `build` job reports success (build compiles and all tests pass).
3. **No artifacts or releases are created for PR builds**, so users only
   ever see builds from merged code.
