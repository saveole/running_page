# Sync Scripts Simplification Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Remove all third-party platform sync scripts except Garmin CN/Global and local file import, simplify GitHub Actions workflow and README documentation.

**Architecture:** Delete 21 unnecessary Python sync scripts, update GitHub Actions workflow to remove ~15 conditional steps, and simplify README to focus only on retained sync methods.

**Tech Stack:** Python 3.12+, GitHub Actions YAML, Markdown

---

## File Structure

**Files to delete:**
- `run_page/nike_sync.py`, `run_page/nike_to_strava_sync.py`
- `run_page/strava_sync.py`, `run_page/strava_to_garmin_sync.py`
- `run_page/coros_sync.py`
- `run_page/keep_sync.py`, `run_page/keep_to_strava_sync.py`
- `run_page/endomondo_sync.py`
- `run_page/joyrun_sync.py`
- `run_page/igpsport_sync.py`
- `run_page/oppo_sync.py`
- `run_page/onelap_sync.py`
- `run_page/tulipsport_sync.py`
- `run_page/komoot_sync.py`
- `run_page/codoon_sync.py`
- `run_page/garmin_to_strava_sync.py`
- `run_page/garmin_sync_cn_global.py`
- `run_page/tcx_to_garmin_sync.py`
- `run_page/tcx_to_strava_sync.py`
- `run_page/gpx_to_strava_sync.py`
- `run_page/garmin_device_adaptor.py`

**Files to modify:**
- `.github/workflows/run_data_sync.yml` - Remove platform-specific sync steps
- `README.md` - Remove platform-specific documentation
- `pyproject.toml` - Remove unused dependencies

---

### Task 1: Delete Third-Party Platform Sync Scripts

**Files:**
- Delete: 21 Python scripts in `run_page/` directory

- [ ] **Step 1: Create a backup branch (safety measure)**

```bash
git checkout -b backup-before-sync-cleanup
git push origin backup-before-sync-cleanup
git checkout saveole
```

- [ ] **Step 2: Delete Nike sync scripts**

```bash
cd /home/ant/projects/running_page
git rm run_page/nike_sync.py run_page/nike_to_strava_sync.py
```

Expected: Files removed from git index

- [ ] **Step 3: Delete Strava sync scripts**

```bash
git rm run_page/strava_sync.py run_page/strava_to_garmin_sync.py
```

- [ ] **Step 4: Delete Coros sync script**

```bash
git rm run_page/coros_sync.py
```

- [ ] **Step 5: Delete Keep sync scripts**

```bash
git rm run_page/keep_sync.py run_page/keep_to_strava_sync.py
```

- [ ] **Step 6: Delete remaining third-party sync scripts**

```bash
git rm run_page/endomondo_sync.py run_page/joyrun_sync.py run_page/igpsport_sync.py
git rm run_page/oppo_sync.py run_page/onelap_sync.py run_page/tulipsport_sync.py
git rm run_page/komoot_sync.py run_page/codoon_sync.py
```

- [ ] **Step 7: Delete bridge/conversion scripts**

```bash
git rm run_page/garmin_to_strava_sync.py run_page/garmin_sync_cn_global.py
git rm run_page/tcx_to_garmin_sync.py run_page/tcx_to_strava_sync.py
git rm run_page/gpx_to_strava_sync.py run_page/garmin_device_adaptor.py
```

- [ ] **Step 8: Verify all deletions**

```bash
git status
```

Expected: 21 files marked as deleted in git status

- [ ] **Step 9: Commit sync script deletions**

```bash
git commit -m "refactor: remove unused third-party platform sync scripts

- Remove Nike, Strava, Coros, Keep, Endomondo sync scripts
- Remove Joyrun, iGPSPORT, OPPO, OneLap, Tulipsport, Komoot sync scripts
- Remove Codoon sync script
- Remove bridge scripts: *_to_strava*, *_to_garmin*, garmin_sync_cn_global
- Remove garmin_device_adaptor.py

Retains: Garmin CN/Global sync, GPX/TCX/FIT local file import"
```

---

### Task 2: Simplify GitHub Actions Workflow

**Files:**
- Modify: `.github/workflows/run_data_sync.yml`

- [ ] **Step 1: Read current workflow file**

```bash
cat .github/workflows/run_data_sync.yml
```

- [ ] **Step 2: Update RUN_TYPE comment**

Edit line 28, change from:
```yaml
  RUN_TYPE: garmin_cn # support strava/nike/garmin/coros/garmin_cn/garmin_sync_cn_global/keep/only_gpx/only_fit/nike_to_strava/strava_to_garmin/tcx_to_garmin/strava_to_garmin_cn/garmin_to_strava/garmin_to_strava_cn/oppo/db_updater, Please change the 'pass' it to your own
```

To:
```yaml
  RUN_TYPE: garmin_cn # support: garmin_cn/garmin/only_gpx/only_tcx/only_fit/db_updater
```

- [ ] **Step 3: Update push trigger paths**

Edit lines 10-24, change `paths:` section from current list to:
```yaml
    paths:
      - run_page/garmin_sync.py
      - run_page/gen_svg.py
      - run_page/gpx_sync.py
      - run_page/tcx_sync.py
      - run_page/fit_sync.py
      - requirements.txt
```

- [ ] **Step 4: Remove Nike sync step**

Delete lines 93-101 (the "Run sync Nike script" step and its conditional)

- [ ] **Step 5: Remove Nike to Strava sync step**

Delete lines 98-105 (the "Run sync Nike to Strava" step)

- [ ] **Step 6: Remove Keep sync step**

Delete lines 103-109 (the "Run sync Keep script" step)

- [ ] **Step 7: Remove Keep to Strava sync step**

Delete lines 115-122 (the "Run sync Keep_to_strava script" step)

- [ ] **Step 8: Remove Coros sync step**

Delete lines 108-116 (the "Run sync Coros script" step)

- [ ] **Step 9: Simplify Garmin CN step**

Find the "Run sync Garmin CN script" step (around line 136-142) and keep it.

Remove the comment about `--tcx` command - no longer needed.

- [ ] **Step 10: Add Garmin Global step**

Add this step after Garmin CN step (if not already present):
```yaml
      - name: Run sync Garmin script
        if: env.RUN_TYPE == 'garmin'
        run: |
          python run_page/garmin_sync.py ${{ secrets.GARMIN_SECRET_STRING }} --only-run
```

- [ ] **Step 11: Remove Garmin CN to Global step**

Delete the "Run sync Garmin CN to Garmin" step (garmin_sync_cn_global)

- [ ] **Step 12: Remove Strava to Garmin steps**

Delete both "Run sync Strava to Garmin" steps (garmin and garmin_cn variants)

- [ ] **Step 13: Remove Garmin to Strava steps**

Delete both "Run sync Garmin-cn to Strava" and "Run sync Garmin to Strava" steps

- [ ] **Step 14: Remove Tulipsport and Oppo steps**

Delete the Tulipsport and Oppo sync steps

- [ ] **Step 15: Verify YAML syntax**

```bash
# Install yamllint if not available
pip3 install yamllint

# Validate the YAML
yamllint .github/workflows/run_data_sync.yml
```

Expected: No errors (or only minor style warnings)

- [ ] **Step 16: Commit workflow changes**

```bash
git add .github/workflows/run_data_sync.yml
git commit -m "refactor: simplify GitHub Actions workflow

- Update RUN_TYPE to only show supported types
- Simplify push trigger paths to retained scripts only
- Remove sync steps for all third-party platforms
- Retain: Garmin CN, Garmin Global, GPX/TCX/FIT import, db_updater"
```

---

### Task 3: Clean Up Python Dependencies

**Files:**
- Modify: `pyproject.toml`

- [ ] **Step 1: Read current dependencies**

```bash
cat pyproject.toml | grep -A 50 "dependencies ="
```

- [ ] **Step 2: Remove stravalib dependency**

Edit `pyproject.toml`, remove line:
```python
  "stravalib==0.10.4",
```

- [ ] **Step 3: Remove stravaweblib dependency**

Edit `pyproject.toml`, remove line:
```python
  "stravaweblib",
```

- [ ] **Step 4: Verify Python syntax**

```bash
python3 -c "import tomli; tomli.load(open('pyproject.toml', 'rb'))"
```

Or if tomli not available:
```bash
python3 -c "exec(open('pyproject.toml').read())"
```

Expected: No syntax errors

- [ ] **Step 5: Commit dependency cleanup**

```bash
git add pyproject.toml
git commit -m "refactor: remove unused Strava dependencies

- Remove stravalib (Strava API client)
- Remove stravaweblib (Strava web scraping)
- Other dependencies retained for Garmin and file import support"
```

---

### Task 4: Simplify README Documentation

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Read current README**

```bash
cat README.md
```

- [ ] **Step 2: Identify sections to remove**

Look for sections about:
- Nike sync
- Strava sync
- Coros sync
- Keep sync
- Any other third-party platform sync

- [ ] **Step 3: Create simplified README content**

The new README should have this structure:
```markdown
## Note
[Keep existing notes section - lines 1-25]

[Keep project title and description]

## Data Sync

### Garmin China/Global

1. Get Garmin Secret String:

\`\`\`bash
python3 run_page/get_garmin_secret.py <email> <password>
# For China region
python3 run_page/get_garmin_secret.py <email> <password> --is-cn
\`\`\`

2. Add the Secret String to GitHub Secrets: `GARMIN_SECRET_STRING_CN`

3. Configure `RUN_TYPE` in `.github/workflows/run_data_sync.yml`:
   - `garmin_cn` for Garmin China
   - `garmin` for Garmin Global

### Local File Sync

Place your GPX/TCX/FIT files in the corresponding directories:

\`\`\`bash
# For GPX files
python3 run_page/gpx_sync.py

# For TCX files
python3 run_page/tcx_sync.py

# For FIT files
python3 run_page/fit_sync.py
\`\`\`

[Keep the rest of README: frontend development, build, deploy, etc.]
```

- [ ] **Step 4: Edit README.md**

Remove all platform-specific sync sections except Garmin and local file import.

Keep the sections for:
- Notes (existing)
- Project title/description
- Garmin sync instructions
- Local file sync instructions (GPX/TCX/FIT)
- Frontend development (npm/pnpm commands)
- Build and deployment
- Docker (if present)
- Contributing

- [ ] **Step 5: Verify Markdown syntax**

```bash
# Optional: use a markdown linter
npm install -g markdownlint-cli
markdownlint README.md
```

- [ ] **Step 6: Commit README changes**

```bash
git add README.md
git commit -m "docs: simplify README to retained sync methods only

- Remove Nike, Strava, Coros, Keep, and other platform sync docs
- Keep Garmin CN/Global sync instructions
- Keep local file sync (GPX/TCX/FIT) instructions
- Keep frontend development and deployment sections"
```

---

### Task 5: Final Verification

- [ ] **Step 1: Verify git log**

```bash
git log --oneline -5
```

Expected: 4-5 commits for this refactoring

- [ ] **Step 2: Verify no deleted files remain**

```bash
git ls-files | grep -E "(nike_sync|strava_sync|coros_sync|keep_sync|endomondo|joyrun|igpsport|oppo|onelap|tulipsport|komoot|codoon|_to_strava|_to_garmin|garmin_sync_cn_global|garmin_device_adaptor)"
```

Expected: No results (all deleted files removed)

- [ ] **Step 3: Count retained sync scripts**

```bash
ls -la run_page/*_sync.py
```

Expected: Only garmin_sync.py, gpx_sync.py, tcx_sync.py, fit_sync.py

- [ ] **Step 4: Verify workflow file is valid**

```bash
cat .github/workflows/run_data_sync.yml | grep "if: env.RUN_TYPE"
```

Expected: Only conditions for garmin_cn, garmin, only_gpx, only_tcx, only_fit, db_updater

- [ ] **Step 5: Optional - Create summary commit**

```bash
# To squash all commits into one (optional, git rebase -i)
# Or create a tag for easy rollback
git tag -a sync-cleanup-$(date +%Y%m%d) -m "Sync scripts cleanup checkpoint"
```

- [ ] **Step 6: Push changes (when ready)**

```bash
git push origin saveole
```

---

## Testing Strategy

Since this is primarily a deletion/refactoring task:

1. **Git history preserved**: All deleted files can be recovered from backup branch or git history
2. **Workflow validation**: YAML syntax checked with yamllint
3. **Incremental commits**: Each task committed separately for easy rollback
4. **Backup branch**: Created at start for safety

## Rollback Plan

If issues arise:

```bash
# Option 1: Revert all commits
git revert HEAD~4..HEAD

# Option 2: Reset to backup branch
git reset --hard backup-before-sync-cleanup

# Option 3: Recover specific deleted file
git checkout backup-before-sync-cleanup -- run_page/filename.py
```

## Dependencies Cleaned

| Package | Purpose | Safe to Remove |
|---------|---------|----------------|
| stravalib | Strava API client | Yes - no Strava sync |
| stravaweblib | Strava web scraping | Yes - no Strava sync |

## Files Summary

| Action | Count |
|--------|-------|
| Files deleted | 21 Python sync scripts |
| Files modified | 3 (workflow, pyproject.toml, README) |
| New files | 0 |
