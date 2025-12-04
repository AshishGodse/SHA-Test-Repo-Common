# SHA-Test-Repo-Common

A centralized GitHub Actions workflow repository that automatically manages and synchronizes reusable workflows across multiple repositories in your organization.

## 🎯 Purpose

This repository implements an automated workflow management system that:
- Hosts reusable GitHub Actions workflows
- Automatically detects changes to reusable workflows
- Propagates updates to all dependent repositories
- Auto-approves and merges update PRs without manual intervention

## 📋 Table of Contents

- [Architecture Overview](#architecture-overview)
- [Workflows](#workflows)
  - [Workflow X - Reusable Workflow](#workflow-x---reusable-workflow)
  - [Workflow Z - Auto Update Engine](#workflow-z---auto-update-engine)
  - [Workflow A - Auto Approve & Merge](#workflow-a---auto-approve--merge)
- [How It Works](#how-it-works)
- [Setup Requirements](#setup-requirements)
- [Usage](#usage)
- [Configuration](#configuration)
- [Security](#security)

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│  Developer Updates workflow-X.yml on main               │
└─────────────────┬───────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────┐
│  WORKFLOW Z: Auto Update Workflow X References          │
│  • Detects change and gets new commit hash              │
│  • Scans organization for SHA-Test* repositories        │
│  • Updates workflow references in all repos             │
│  • Creates PRs with "Auto Approve:" prefix              │
└─────────────────┬───────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────┐
│  WORKFLOW A: Auto Approve and Merge                     │
│  • Triggered when Workflow Z completes                  │
│  • Finds PRs with "Auto Approve" in title               │
│  • Approves and merges PRs automatically                │
│  • Deletes branches after merge                         │
└─────────────────┬───────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────────────────┐
│  ✅ All dependent repos updated automatically!          │
└─────────────────────────────────────────────────────────┘
```

## 🔄 Workflows

### Workflow X - Reusable Workflow

**File:** `.github/workflows/workflow-X.yml`

A reusable workflow that can be called by other repositories. This is the template workflow that gets referenced by dependent repositories.

**Usage in dependent repos:**
```yaml
jobs:
  call-workflow-x:
    uses: AshishGodse/SHA-Test-Repo-Common/.github/workflows/workflow-X.yml@<commit-hash>
    with:
      environment: production
```

**Key Features:**
- Accepts environment input parameter
- Can be referenced by commit hash for version control
- Centralized logic that applies to multiple repositories

---

### Workflow Z - Auto Update Engine

**File:** `.github/workflows/workflow-Z.yml`

The automation engine that detects changes to Workflow X and propagates them to all dependent repositories.

**Triggers:**
- Automatically when `workflow-X.yml` is modified on main branch
- Manually via `workflow_dispatch` with optional organization parameter

**What It Does:**
1. ✅ Detects changes to workflow-X.yml and captures the new commit hash
2. ✅ Fetches all repositories from the organization
3. ✅ Filters for repositories starting with "SHA-Test"
4. ✅ Clones each repository and scans ALL workflow files
5. ✅ Finds references to `workflow-X.yml@<old-hash>`
6. ✅ Replaces old hash with new commit hash
7. ✅ Creates a branch: `update-workflow-x-<short-hash>`
8. ✅ Commits with message: `Auto Approve: update workflow-x reference to <hash>`
9. ✅ Creates Pull Request in each affected repository

**Example Output:**
```
Processing SHA-Test-Repo-1:
  ✅ Found: workflow-X.yml@abc123old in .github/workflows/workflow.yml
  ✅ Updated to: workflow-X.yml@def789new
  ✅ Created branch: update-workflow-x-def789ne
  ✅ Created PR #42
```

---

### Workflow A - Auto Approve & Merge

**File:** `.github/workflows/workflow-A.yml`

The automation workflow that approves and merges PRs created by Workflow Z.

**Triggers:**
- Automatically when Workflow Z completes successfully

**What It Does:**
1. ✅ Waits for Workflow Z to complete successfully
2. ✅ Fetches all "SHA-Test*" repositories
3. ✅ Searches for open PRs with "Auto Approve" in the title
4. ✅ Approves PRs using `WORKFLOW_A_TOKEN`
5. ✅ Merges PRs using `WORKFLOW_Z_TOKEN` (squash merge)
6. ✅ Deletes branches after successful merge
7. ✅ Provides summary of approved and merged PRs

**Example Output:**
```
📊 FINAL SUMMARY
✅ Total PRs approved: 3
✅ Total PRs merged: 3
```

---

## 🚀 How It Works

### Step-by-Step Process

**1. Developer Makes a Change**
```bash
# Developer updates workflow-X.yml
git add .github/workflows/workflow-X.yml
git commit -m "Update workflow logic"
git push origin main
```

**2. Workflow Z Activates (2-3 minutes)**
- Detects the push to workflow-X.yml
- Gets new commit hash: `def789ghi456`
- Scans organization for repositories starting with "SHA-Test"
- For each repository:
  - Clones the repo
  - Searches all workflow files for `workflow-X.yml@<hash>`
  - Updates references to new hash
  - Creates PR with "Auto Approve:" prefix

**3. Workflow A Activates (1-2 minutes)**
- Triggered by Workflow Z completion
- Finds all PRs with "Auto Approve" in title
- Approves each PR
- Merges each PR automatically
- Cleans up branches

**Total Time:** ~5 minutes for complete automated rollout! ⚡

---

## ⚙️ Setup Requirements

### 1. Required Secrets

You need two GitHub Personal Access Tokens (PATs) with the following permissions:

**`WORKFLOW_Z_TOKEN`:**
- `repo` (full control)
- `workflow` (update workflows)
- Used for: Creating branches, pushing commits, creating PRs, merging PRs

**`WORKFLOW_A_TOKEN`:**
- `repo` (full control)
- Used for: Approving PRs (must be different from WORKFLOW_Z_TOKEN)

> **Why two tokens?** GitHub security requires that PR approval and merge must be done by different actors.

### 2. Repository Settings

Ensure the following settings are enabled in **all dependent repositories**:
- ✅ Allow GitHub Actions to create and approve pull requests
- ✅ Allow auto-merge
- ✅ Require approval before merging (satisfied by Workflow A)

### 3. Repository Naming Convention

Repositories must start with `SHA-Test` to be automatically processed:
- ✅ `SHA-Test-Repo-1`
- ✅ `SHA-Test-Repo-2`
- ✅ `SHA-Test-API`
- ❌ `Test-SHA-Repo` (will be ignored)

---

## 📖 Usage

### For Developers

**To Update the Reusable Workflow:**
```bash
# 1. Make changes to workflow-X.yml
vim .github/workflows/workflow-X.yml

# 2. Commit and push to main
git add .github/workflows/workflow-X.yml
git commit -m "feat: update workflow logic"
git push origin main

# 3. Wait ~5 minutes - automation handles the rest! ☕
```

**To Manually Trigger Updates:**
1. Go to Actions tab
2. Select "Workflow Z - Auto Update Workflow X References"
3. Click "Run workflow"
4. (Optional) Specify target organization

### For Repository Maintainers

**To Add a New Repository:**
1. Create repository with name starting with `SHA-Test`
2. Add workflow that references workflow-X.yml:
```yaml
jobs:
  my-job:
    uses: AshishGodse/SHA-Test-Repo-Common/.github/workflows/workflow-X.yml@<commit-hash>
    with:
      environment: production
```
3. The repository will automatically be included in future updates!

**To Remove a Repository:**
1. Rename the repository (remove `SHA-Test` prefix)
2. Or delete the workflow-X.yml reference from workflow files

---

## 📝 Configuration

### Repository Configuration File

**File:** `.github/repos-config.yml`

This file documents which repositories reference workflow-X (for reference purposes):

```yaml
repos:
  - name: SHA-Test-Repo-1
    owner: AshishGodse
    workflow_file: .github/workflows/workflow.yml
  - name: SHA-Test-Repo-2
    owner: AshishGodse
    workflow_file: .github/workflows/workflow.yml
  - name: SHA-Test-Repo-3
    owner: AshishGodse
    workflow_file: .github/workflows/workflow.yml
```

> **Note:** This file is for documentation only. Workflow Z dynamically discovers repositories, so this file doesn't need to be updated when adding new repos.

---

## 🔒 Security

### Commit Hash Pinning

Dependent repositories reference workflows by commit hash instead of branch:
```yaml
# ✅ Good - pinned to specific version
uses: owner/repo/.github/workflows/workflow.yml@abc123def456

# ❌ Avoid - always uses latest (unpredictable)
uses: owner/repo/.github/workflows/workflow.yml@main
```

**Benefits:**
- ✅ Predictable behavior
- ✅ No surprise breaking changes
- ✅ Controlled rollout of updates
- ✅ Easy rollback by reverting commit

### Token Security

- Tokens are stored as repository secrets
- Two separate tokens prevent excessive privileges
- Tokens have minimum required permissions
- Workflow runs are logged and auditable

### Pull Request Safety

- All changes go through PR process
- Clear "Auto Approve:" prefix identifies automated PRs
- Manual PRs are never touched by automation
- Branch protection rules still apply

---

## 🐛 Troubleshooting

### Workflow Z didn't create PRs

**Check:**
1. Does workflow-X.yml actually exist in dependent repos?
2. Are the hash references in correct format: `workflow-X.yml@<40-char-hash>`?
3. Check Workflow Z logs for errors

### Workflow A didn't merge PRs

**Check:**
1. Did Workflow Z complete successfully?
2. Do PRs have "Auto Approve" in the title?
3. Are both tokens configured correctly?
4. Check repository settings allow auto-merge

### Some repositories were skipped

**Check:**
1. Does repository name start with "SHA-Test"?
2. Does repository have `.github/workflows` directory?
3. Do workflow files contain `workflow-X.yml@<hash>` references?

---

## 📊 Monitoring

### View Workflow Runs

1. Go to the **Actions** tab
2. Select the workflow you want to monitor
3. Click on a specific run to see detailed logs

### Key Metrics

- **Workflow Z Duration:** ~2-3 minutes
- **Workflow A Duration:** ~1-2 minutes
- **Total Automation Time:** ~5 minutes

---

## 🤝 Contributing

1. Make changes to workflow files
2. Test in a separate branch first
3. Once verified, merge to main
4. Automation will handle the rest!

---

## 📄 License

This is a Proof of Concept (PoC) repository for automated workflow management.

---

## 🆘 Support

For issues or questions, please contact the DevOps team or create an issue in this repository.

---

**Last Updated:** December 4, 2025