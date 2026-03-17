# Sync Scripts Simplification Design

**Date:** 2026-03-17
**Status:** Approved
**Author:** Claude

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
# 1. Delete unused sync scripts
rm run_page/nike_sync.py run_page/nike_to_strava_sync.py
rm run_page/strava_sync.py run_page/strava_to_garmin_sync.py
rm run_page/coros_sync.py
rm run_page/keep_sync.py run_page/keep_to_strava_sync.py
rm run_page/endomondo_sync.py
rm run_page/joyrun_sync.py
rm run_page/igpsport_sync.py
rm run_page/oppo_sync.py
rm run_page/onelap_sync.py
rm run_page/tulipsport_sync.py
rm run_page/komoot_sync.py
rm run_page/codoon_sync.py
rm run_page/garmin_to_strava_sync.py
rm run_page/garmin_sync_cn_global.py
rm run_page/tcx_to_garmin_sync.py
rm run_page/tcx_to_strava_sync.py
rm run_page/gpx_to_strava_sync.py
rm run_page/garmin_device_adaptor.py

# 2. Update .github/workflows/run_data_sync.yml

# 3. Update README.md

# 4. Commit changes
```

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
