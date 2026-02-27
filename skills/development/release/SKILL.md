---
name: release
description: Prepare and publish a new cryptobot-python release. Use when the user asks to cut a release, bump the package version, update HISTORY.md, create a git tag, or create a GitHub release.
---

# Release Skill

Create a new release for the cryptobot-python package.

## Usage

```
/release <version>
```

Where `<version>` is the new semantic version (e.g., `0.4.2`, `0.5.0`, `1.0.0`).

## Instructions

When the user invokes this skill with a version number, perform the following steps:

1. **Validate the version**: Ensure the version follows semantic versioning (X.Y.Z format).

2. **Update version files**:
   - Update `version` in `pyproject.toml`
   - Update `__version__` in `cryptobot/__init__.py`

3. **Ask for release notes**: Ask the user what changes should be included in the release notes.

4. **Update HISTORY.md**: Add a new entry at the top of the History section with:
   - Version number and today's date
   - The release notes provided by the user

5. **Run tests**: Execute `make test` to ensure all tests pass before proceeding.

6. **Create commit**: Create a commit with message `chore(release): prepare v<version> release`

7. **Create tag**: Create an annotated git tag `v<version>` pointing to the release commit.

8. **Create GitHub release**: Use `gh release create` to create a GitHub release with:
   - Tag: `v<version>`
   - Title: `v<version>`
   - Release notes from the HISTORY.md entry

9. **Report completion**: Provide the user with:
   - Summary of files changed
   - Link to the GitHub release
