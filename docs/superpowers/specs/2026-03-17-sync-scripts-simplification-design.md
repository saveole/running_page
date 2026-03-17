# Sync Scripts Simplification Design

**Date:** 2026-03-17
**Status:** Spec Reviewed - Approved
**Author:** Claude
**Reviewer:** Spec Review Agent

## Overview

Simplify the running_page project by removing all third-party platform sync scripts except Garmin CN/Global and local file import (GPX/TCX/FIT). This reduces maintenance burden, improves security, and makes the codebase clearer.

## Goals

1. Keep only: Garmin CN sync, Garmin Global sync, and local file import (GPX/TCX/FIT)
2. Remove all 20+ third-party platform sync scripts
3. Simplify GitHub Actions workflow
4. Simplify README documentation
5. Preserve git history (delete files only from current branch)

## Files to Delete (21 total)

### Third-Party Platform Sync Scripts (13)

- `nike_sync.py`, `nike_to_strava_sync.py`
- `strava_sync.py`, `strava_to_garmin_sync.py`
- `coros_sync.py`
- `keep_sync.py`, `keep_to_strava_sync.py`
- `endomondo_sync.py`
- `joyrun_sync.py`
- `igpsport_sync.py`
- `oppo_sync.py`
- `onelap_sync.py`
- `tulipsport_sync.py`
- `komoot_sync.py`
- `codoon_sync.py`

### Bridge/Conversion Scripts (7)

- `garmin_to_strava_sync.py`
- `garmin_sync_cn_global.py`
- `tcx_to_garmin_sync.py`
- `tcx_to_strava_sync.py`
- `gpx_to_strava_sync.py`
- `garmin_device_adaptor.py`

## Files to Retain

```
run_page/
├── garmin_sync.py          # Garmin CN/Global sync
├── gpx_sync.py             # Local GPX import
├── tcx_sync.py             # Local TCX import
├── fit_sync.py             # Local FIT import
├── gen_svg.py              # SVG generation
├── db_updater.py           # Database update utility
├── auto_share_sync.py      # Auto share utility
├── utils.py                # Utility functions
├── config.py               # Configuration
└── generator.py            # Data generator
```

The `gpxtrackposter/` directory (SVG drawing core) is fully preserved.

## GitHub Actions Workflow Changes

**File:** `.github/workflows/run_data_sync.yml`

**Supported RUN_TYPE values after simplification:**
- `garmin_cn` - Garmin China sync
- `garmin` - Garmin Global sync
- `only_gpx` - Local GPX file sync
- `only_tcx` - Local TCX file sync
- `only_fit` - Local FIT file sync
- `db_updater` - Database schema updates

**Workflow push trigger paths simplified to:**
```yaml
paths:
  - run_page/garmin_sync.py
  - run_page/gen_svg.py
  - run_page/gpx_sync.py
  - run_page/tcx_sync.py
  - run_page/fit_sync.py
  - requirements.txt
```

**Remove:** All conditional steps for third-party platforms (~15+ `if` conditions)

## README Simplification

**New structure:**
1. Project introduction
2. Garmin sync instructions (CN + Global)
3. Local file sync instructions (GPX/TCX/FIT)
4. Local development (frontend)
5. Build and deploy instructions

**Remove:** All platform-specific sync docs for Nike, Strava, Coros, Keep, etc.

## Implementation Steps

```bash
# 1. Delete unused sync scripts (21 files)
cd run_page
rm nike_sync.py nike_to_strava_sync.py
rm strava_sync.py strava_to_garmin_sync.py
rm coros_sync.py
rm keep_sync.py keep_to_strava_sync.py
rm endomondo_sync.py
rm joyrun_sync.py
rm igpsport_sync.py
rm oppo_sync.py
rm onelap_sync.py
rm tulipsport_sync.py
rm komoot_sync.py
rm codoon_sync.py
rm garmin_to_strava_sync.py
rm garmin_sync_cn_global.py
rm tcx_to_garmin_sync.py
rm tcx_to_strava_sync.py
rm gpx_to_strava_sync.py
rm garmin_device_adaptor.py

# 2. Update .github/workflows/run_data_sync.yml
#    - Update env.RUN_TYPE comment to list only: garmin_cn/garmin/only_gpx/only_tcx/only_fit/db_updater
#    - Update paths trigger (lines ~10-24) to only include retained scripts
#    - Remove sync steps for: nike, nike_to_strava, strava, coros, keep, keep_to_strava,
#      garmin, garmin_cn (re-add with simplified form), garmin_sync_cn_global,
#      only_gpx (re-add), only_tcx (re-add), strava_to_garmin, strava_to_garmin_cn,
#      garmin_to_strava, garmin_to_strava_cn, tulipsport, oppo, db_updater (re-add)
#    - Keep SVG generation steps unchanged

# 3. Update README.md
#    - Remove sections: Nike sync, Strava sync, Coros sync, Keep sync, and other platforms
#    - Keep sections: Garmin sync (CN + Global), local file sync (GPX/TCX/FIT), frontend development
#    - Update RUN_TYPE comment in workflow example

# 4. Commit changes
git add -A
git commit -m "Simplify: Remove unused platform sync scripts"
```

## Dependency Cleanup

**Remove from `pyproject.toml`:**
- `stravalib==0.10.4` - Strava API client
- `stravaweblib` - Strava web scraping

**Keep:** All other dependencies (used by Garmin sync, GPX/TCX/FIT parsing, SVG generation)

## Optional Follow-up

Evaluate `pyproject.toml` dependencies and remove packages no longer needed:
- `stravalib`
- `stravaweblib`
- (other platform-specific libraries)

## Trade-offs

**Benefits:**
- Cleaner codebase, easier to understand
- Reduced maintenance burden
- Fewer security concerns (third-party API credentials)
- Faster CI workflow

**Drawbacks:**
- Loss of integration with other platforms
- Git history contains deleted files (but still accessible)
