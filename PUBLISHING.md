# Publishing to PyPI

This document describes how to set up automatic publishing to PyPI.

## Prerequisites

1. **PyPI Account**: Create account at https://pypi.org/account/register/
2. **Package Name**: Reserve the name `claude-explore` on PyPI

## Setup Options

### Option 1: Trusted Publishing (Recommended - No Tokens!)

This is the modern, secure approach that doesn't require storing API tokens.

#### Steps:

1. **Create the package on PyPI** (first time only):
   ```bash
   # Build locally
   uv build

   # Publish manually for first release
   uv publish
   # Enter your PyPI username and password when prompted
   ```

2. **Configure Trusted Publisher on PyPI**:
   - Go to https://pypi.org/manage/project/claude-explore/settings/
   - Scroll to "Publishing" section
   - Click "Add a new publisher"
   - Fill in:
     - **PyPI Project Name**: `claude-explore`
     - **Owner**: `<your-github-username>` (e.g., `akatz-ai`)
     - **Repository name**: `claude-explore` (or whatever your repo is named)
     - **Workflow name**: `publish.yml`
     - **Environment name**: (leave blank)
   - Click "Add"

3. **Done!** Future pushes to `main` will automatically publish to PyPI (no tokens needed)

### Option 2: API Token (Alternative)

If you prefer using API tokens instead:

#### Steps:

1. **Create PyPI API Token**:
   - Go to https://pypi.org/manage/account/token/
   - Click "Add API token"
   - Token name: `github-actions-claude-explore`
   - Scope: "Project: claude-explore" (or "Entire account" for first publish)
   - Copy the token (starts with `pypi-`)

2. **Add Token to GitHub Secrets**:
   - Go to your GitHub repo → Settings → Secrets and variables → Actions
   - Click "New repository secret"
   - Name: `PYPI_API_TOKEN`
   - Value: Paste the token from step 1
   - Click "Add secret"

3. **Done!** The workflow will use this token for publishing

## Workflow Behavior

The `.github/workflows/publish.yml` workflow:

- **Triggers on**:
  - Pushes to `main` branch (when `pyproject.toml` or `src/` changes)
  - Manual workflow dispatch (via GitHub Actions UI)

- **What it does**:
  1. Checks out code
  2. Sets up Python 3.12 and UV
  3. Runs tests (must pass)
  4. Builds the package
  5. Publishes to PyPI

- **Safeguards**:
  - Only triggers on specific file changes (not every push)
  - Runs tests before publishing
  - Skips if version already exists on PyPI

## Version Management

**IMPORTANT**: Update version in `pyproject.toml` before pushing to trigger publish:

```toml
[project]
name = "claude-explore"
version = "0.1.1"  # <- Increment this
```

PyPI doesn't allow re-uploading the same version, so:
- `0.1.0` → First release
- `0.1.1` → Bug fix
- `0.2.0` → New features
- `1.0.0` → Stable release

## Manual Publishing (Local)

If you want to publish manually from your machine:

```bash
# Build
uv build

# Publish to PyPI
uv publish

# Or publish to TestPyPI first
uv publish --publish-url https://test.pypi.org/legacy/
```

## First-Time Publishing Checklist

Before your first publish:

- [ ] PyPI account created
- [ ] Package name `claude-explore` available/reserved
- [ ] Version in `pyproject.toml` is correct (start with `0.1.0`)
- [ ] Tests pass: `uv run pytest tests/`
- [ ] README.md is complete
- [ ] LICENSE file exists
- [ ] Either:
  - [ ] Trusted publisher configured on PyPI, OR
  - [ ] `PYPI_API_TOKEN` secret added to GitHub

## Testing the Workflow

1. **Test on TestPyPI first** (optional but recommended):
   ```bash
   # Modify workflow to use TestPyPI
   # publish-url: https://test.pypi.org/legacy/

   # Push and verify it works
   git push origin main

   # Test installing from TestPyPI
   pip install --index-url https://test.pypi.org/simple/ claude-explore
   ```

2. **Publish to real PyPI**:
   ```bash
   # Increment version in pyproject.toml
   # Push to main
   git add pyproject.toml
   git commit -m "Bump version to 0.1.0"
   git push origin main

   # Check GitHub Actions tab to see workflow progress
   ```

3. **Verify**:
   ```bash
   # Should work immediately after publish
   pip install claude-explore
   claude-explore --version
   ```

## Troubleshooting

### "Package already exists"
- Version already published. Increment version in `pyproject.toml`

### "Invalid or expired token"
- Regenerate PyPI token and update GitHub secret

### "Trusted publisher not configured"
- Follow Option 1 steps to configure trusted publishing on PyPI

### Tests fail in workflow
- Run `uv run pytest tests/` locally to debug
- Fix tests before pushing

## GitHub Actions Badge

Add to README.md:

```markdown
[![PyPI](https://img.shields.io/pypi/v/claude-explore)](https://pypi.org/project/claude-explore/)
[![Tests](https://github.com/<your-username>/claude-explore/workflows/Run%20Tests/badge.svg)](https://github.com/<your-username>/claude-explore/actions)
```
