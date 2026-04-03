# CI Workflows Guide

This folder contains ETL GitHub Actions workflows for compiler checks, syntax checks, generated-header validation, and release publishing.

## Workflow Inventory

- `msvc.yml`: Windows/MSVC matrix checks.
- `gcc.yml`: Linux/GCC matrix checks.
- `clang.yml`: Linux and macOS/Clang matrix checks.
- `syntax-checks.yml`: Header syntax validation matrix.
- `generator.yml`: Generated-header drift checks.
- `platformio-update.yml`: Publish package to PlatformIO on release.

## CI Baseline Rules

Apply these defaults to all workflows unless there is a documented exception.

- Add `concurrency` with `cancel-in-progress: true`.
- Set explicit `timeout-minutes` per job.
- Pin runner images (avoid `*-latest` unless intentionally required).
- Use least-privilege `permissions`.
- Pin externally installed tool versions when practical.

## Cache Key Convention

Build workflows use this key pattern for ccache isolation and invalidation:

`<runner.os>-<github.workflow>-<github.job>-<matrix-axes>-<hashFiles(CMakeLists.txt,test/CMakeLists.txt)>`

Rationale:

- Isolates caches between workflows and jobs.
- Separates matrix variants to avoid cross-pollution.
- Invalidates cache when build graph definitions change.

## Sanitizer Policy

Current GCC/Clang CI sets:

`ASAN_OPTIONS=alloc_dealloc_mismatch=0,detect_leaks=0`

Rationale:

- Keeps CI signal stable while historical sanitizer noise is being addressed.
- If changed, update both GCC and Clang workflows together.

## Failure Diagnostics

Compiler workflows upload failure artifacts when a job fails.

Typical artifact contents:

- `test/etl_tests` or `test/etl_tests.exe`
- `CMakeCache.txt`
- `Testing/Temporary/LastTest.log`

Retention is currently 14 days.

## Trigger Notes

Compiler and syntax workflows run on:

- Push to `master`, `development`, and `pull-request/*`.
- Pull requests with events: `opened`, `synchronize`, `reopened`.

Generator checks intentionally use a narrower PR branch filter than compiler checks.

## Change Checklist

When adding or modifying workflow jobs:

1. Keep timeout, concurrency, and permissions explicit.
2. Keep cache key format consistent.
3. Ensure compiler matrix changes preserve intended coverage.
4. Update this README if conventions change.
5. Verify with actionlint and a PR dry run.
