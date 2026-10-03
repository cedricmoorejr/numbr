# Releasing numbr

Only maintainers should perform a release. PyPI releases are immutable, so the
version, tag, and artifacts must be verified before publishing the GitHub
release.

## One-time repository setup

1. In GitHub, create an environment named `pypi`. Restrict deployments to
   protected tags matching `v*` and require maintainer approval when the plan
   supports it.
2. In the PyPI `numbr` project, add a GitHub Actions Trusted Publisher with:

   - owner: `doydl-technologies`
   - repository: `numbr`
   - workflow: `publish.yml`
   - environment: `pypi`

3. Protect `main` and release tags. Require the CI and CodeQL checks before
   merging release changes.

The publishing workflow uses short-lived OpenID Connect credentials. Do not add
a long-lived PyPI API token to the repository.

## Prepare a release

1. Choose the next version using Semantic Versioning.
2. Set `project.version` in `pyproject.toml`.
3. Move the relevant entries from `Unreleased` into a dated version section in
   `CHANGELOG.md`, and update its comparison links.
4. Run the complete release gate from a clean virtual environment:

   ```bash
   python -m pip install --upgrade pip
   python -m pip install -e '.[dev]'
   ruff check .
   ruff format --check .
   pytest
   rm -rf build dist
   python -m build
   python -m twine check dist/*
   ```

5. Install the wheel into a separate virtual environment and run a smoke test:

   ```bash
   python -m venv .release-venv
   source .release-venv/Scripts/activate
   python -m pip install dist/*.whl
   cd /tmp
   python -I -c "import numbr; assert numbr.__version__ == '2.1.1'; assert numbr.Cast('Ⅳ', 'Ordinal Word') == 'fourth'"
   ```

6. Review the wheel and source archive contents. Commit the release changes and
   merge them into `main` only after all required checks pass.

## Publish 2.1.1

1. On GitHub, draft a new release targeting `main`.
2. Create tag `v2.1.1` and use release title `numbr 2.1.1`.
3. Generate the release notes, then reconcile them with the `2.1.1` section in
   `CHANGELOG.md`.
4. Save a draft if more review is needed. Publishing the release is the point
   of no return: it triggers `.github/workflows/publish.yml`.
5. Confirm that the workflow built both `numbr-2.1.1.tar.gz` and
   `numbr-2.1.1-py3-none-any.whl`, validated the tag/version match, and published
   through the `pypi` environment.
6. Confirm the new release on PyPI and install it in a fresh environment:

   ```bash
   python -m pip install --no-cache-dir numbr==2.1.1
   python -c "import numbr; print(numbr.__version__)"
   ```

If publishing fails, fix the workflow or configuration and rerun the failed
job. Never reuse a version already accepted by PyPI.
