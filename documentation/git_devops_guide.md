```
#!/bin/bash
# =====================================================================
# GCP MarTech Analytics Warehouse - Git DevOps Guide
# This file defines the best practices and safe workflow for syncing
# code from Dev → Prod repositories.
# =====================================================================

# ---------------------------------------------------------------------
# STEP 0 — Overview
# ---------------------------------------------------------------------
# - Dev repo: martechanalyticsdev
# - Prod repo: martechanalyticsprod
# - Branch: master (both sides)
# - Goal: Promote tested and verified code from Dev → Prod safely.
#
# This guide prevents commit loss, merge conflicts, and history drift.
# ---------------------------------------------------------------------

# ---------------------------------------------------------------------
# STEP 1 — Commit & Push All Changes to Dev Repo
# ---------------------------------------------------------------------
# 1. Work on your Dev repo as usual.
# 2. Stage, commit, and push to Dev remote.
#
# Example:
#   git add .
#   git commit -m "V1.0.2 — Enhanced Google Ads ETL logic" (you can use PyCharm UI for this)
#   git push martechanalyticsdev master (you can use PyCharm UI for this)

# HIGHLY IMPORTANT: ***** Always go through the github repo and the associated folders / files to check commit mesaage
# and you should adopt to well structured commit message practice which has been already implemented. DONT BREAK IT.
# ---------------------------------------------------------------------

# ---------------------------------------------------------------------
# STEP 2 — Sync Dev → Prod
# ---------------------------------------------------------------------
# From your Prod repo directory:

echo "🔄 Fetching latest from Dev and Prod..."
git fetch martechanalyticsdev
git fetch martechanalyticsprod

# ---------------------------------------------------------------------
# STEP 2.1 — Check Differences (for awareness)
# ---------------------------------------------------------------------
echo "Comparing Dev vs Prod..."
git diff martechanalyticsdev/master martechanalyticsprod/master --stat

# ---------------------------------------------------------------------
# STEP 2.2 — Merge Dev changes into Prod
# ---------------------------------------------------------------------
echo "Merging Dev → Prod..."
git merge martechanalyticsdev/master --allow-unrelated-histories || true

(OR)

git merge martechanalyticsdev/master --allow-unrelated-histories

# If README.md conflict appears, keep the Dev version automatically
if git diff --name-only --diff-filter=U | grep -q "README.md"; then
  echo "README conflict detected — keeping Dev version"
  git checkout --ours README.md
  git add README.md
  git commit -m "Merge: resolved README conflict — kept Dev version"
else
  echo "No merge conflicts detected."
fi

# ---------------------------------------------------------------------
# STEP 2.3 — Push merged result to Prod
# ---------------------------------------------------------------------
echo "Pushing to Prod..."
git push martechanalyticsprod master

(OR)

git push martechanalyticsprod master --force-with-lease

# ---------------------------------------------------------------------
# STEP 3 — Verify Sync
# ---------------------------------------------------------------------
echo "Verifying Dev ↔ Prod alignment..."
git fetch --all
git diff martechanalyticsdev/master martechanalyticsprod/master --stat

# ---------------------------------------------------------------------
# STEP 4 — Tag the Release
# ---------------------------------------------------------------------
# After successful promotion, tag the Prod release.
# Example:
#   git tag -a v1.0.2 -m "Promoted stable version to Prod"
#   git push martechanalyticsprod v1.0.2
# ---------------------------------------------------------------------

# ---------------------------------------------------------------------
# BEST PRACTICES
# ---------------------------------------------------------------------
# ✅ NEVER touch PROD / NEVER do changes in PROD.
# ✅ Always push Dev → Dev first before syncing to Prod.
# ✅ Use descriptive semantic commit messages:
#      V1.0.0 — Historical Backfill
#      V1.0.1 — Incremental Loading
#      V1.1.0 — Dev/Prod DAGs Enhancement
# ✅ Pull before push if Prod is ahead (avoid rejected pushes).
# ✅ Tag every stable Prod release.
# ✅ Avoid 'git push --force' unless absolutely necessary.
# ✅ Never rebase after pushing to shared or Prod branch.
# ✅ Keep commits clean, avoid “Commit 1” or empty commits.
# ✅ Use `git status` and `git log --oneline` often to verify state.
# ✅ Resolve conflicts properly (especially README.md) before committing.
# ✅ Keep .gitignore updated (exclude __pycache__, .idea, .env, etc.)
# ✅ Use “--force-with-lease” instead of plain “--force” for safety.
# ✅ Always verify the final diff before tagging a release.
# ---------------------------------------------------------------------

echo "Dev → Prod sync completed successfully."
echo "Remember to tag the release and verify in GitHub UI."
```