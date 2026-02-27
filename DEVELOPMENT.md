# Development and Release Workflow

This document describes how to develop, build, and release the Shim Executable project: local commands, CI behavior, and release steps.

---

## Overview

- **Development:** Work on a branch, open a PR; every push to the PR runs a Windows build in GitHub Actions.
- **Release:** Update the version in source, tag with `v*`, and push; CI builds and publishes a GitHub Release with Windows artifacts.

---

## Development Workflow

### Local setup

- **OS:** Windows (build uses MSVC).
- **Tools:** Visual Studio (or Build Tools) with C++ and Windows SDK, and **GNU Make**.
- **Environment:** Build from a shell where MSVC is in `PATH` (e.g. *x64 Native Tools Command Prompt for VS*), or run the VS environment script before `make`.

### Local build

From the repository root, with MSVC available:

```bash
nmake
```

**Output:**

- `bin\shim_exec.exe` — main executable  
- `bin\shim_exec.sha256` — SHA-256 checksum of the executable  

Intermediate files (`.obj`, `.res`, shim executables) are removed by the Makefile after the build.

### Branching and pull requests

1. Create a branch from `main` (or `master`):

   ```bash
   git checkout -b feature/my-change
   ```

2. Make changes, build locally with `make`, then commit and push:

   ```bash
   git add .
   git commit -m "Description of change"
   git push -u origin feature/my-change
   ```

3. Open a **Pull Request** targeting `main` or `master`.

4. **CI (Build PR):** Every push to that PR triggers `.github/workflows/build-pr.yml`:
   - Runs on `windows-latest`
   - Installs Make, sets up MSVC, runs `make`
   - Verifies `bin\shim_exec.exe` and `bin\shim_exec.sha256` exist  

   Fix any CI failures before merging.

5. Merge the PR when review and CI are green.

---

## Release Workflow

### Version numbering

- Version is defined in **`include/version.h`** as `VERSION_MAJOR`, `VERSION_MINOR`, and `VERSION_PATCH`.
- **Git tags** must match this (e.g. `2.3.0` in source → tag `v2.3.0`).

### Updating the version

1. Edit `include/version.h` and set:

   ```c
   #define VERSION_MAJOR   X
   #define VERSION_MINOR   Y
   #define VERSION_PATCH   Z
   ```

2. Commit the version bump:

   ```bash
   git add include/version.h
   git commit -m "Bump version to X.Y.Z"
   ```

### Creating a release

1. **Tag** the commit that has the correct version (use the same number as in `version.h`):

   ```bash
   git tag v2.3.0
   ```

2. **Push** the tag to trigger the release workflow:

   ```bash
   git push origin v2.3.0
   ```

3. **CI (Build and Release):** Pushing a tag matching `v*` runs `.github/workflows/release.yml`:
   - **Build job:** Same as PR build (Windows, Make, MSVC); produces `bin/shim_exec.exe` and `bin/shim_exec.sha256`, then uploads them as artifacts.
   - **Release job:** Downloads those artifacts and creates a **GitHub Release** for the tag with:
     - Title: `Release vX.Y.Z`
     - Auto-generated release notes
     - Attached files: `shim_exec.exe`, `shim_exec.sha256`

4. The new release appears under **Releases** on GitHub; users can download the Windows build and checksum from there.

### Release checklist

| Step | Command / action |
|------|-------------------|
| Bump version in source | Edit `include/version.h` (MAJOR.MINOR.PATCH) |
| Commit version bump | `git add include/version.h` then `git commit -m "Bump version to X.Y.Z"` |
| Create tag | `git tag vX.Y.Z` (match version in `version.h`) |
| Publish release | `git push origin vX.Y.Z` |
| Verify | Check the Actions tab and the Releases page on GitHub |

---

## Workflow Files Reference

| File | Trigger | Purpose |
|------|---------|--------|
| `.github/workflows/build-pr.yml` | Push to a PR targeting `main` or `master` | Build on Windows and verify artifacts. |
| `.github/workflows/release.yml` | Push of a tag `v*` (e.g. `v2.3.0`) | Build on Windows, then create GitHub Release and attach `shim_exec.exe` and `shim_exec.sha256`. |

---

## Quick command reference

| Task | Command |
|------|---------|
| Build locally | `make` |
| Version bump | Edit `include/version.h`, then commit. |
| Create and publish release | `git tag vX.Y.Z` then `git push origin vX.Y.Z` |
| List existing tags | `git tag -l` |
| Delete a tag locally | `git tag -d vX.Y.Z` |
| Delete a tag on remote | `git push origin --delete vX.Y.Z` (use with care; prefer fixing forward with a new tag.) |

---

## Notes

- **Tag vs version.h:** Keep the git tag (e.g. `v2.3.0`) and the version in `include/version.h` in sync so the release title and embedded version info match.
- **Default branch:** If your repo uses a different default branch than `main` or `master`, update the `branches` list in `.github/workflows/build-pr.yml` so PRs to that branch run the build.
