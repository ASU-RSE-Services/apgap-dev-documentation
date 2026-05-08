
# Development Workflow

### Branching Strategy

- Development work should branch from and merge back to the `development` branch
- Future plans include dedicated `uat` and `main` branches for UAT and production environments
- Currently, only the `development` environment has an associated deployment

### CI/CD Pipeline

- GitHub Actions automates testing pipelines
- The workflow configuration is in `.github/workflows/ci.yml`
- Tests run in a Docker Compose environment to execute unit tests
- CI checks are triggered on pull requests to `development`, `uat`, and `main` branches

### Semantic Versioning

This repository uses [Python Semantic Release](https://python-semantic-release.readthedocs.io/) to automatically determine and publish version numbers based on commit messages. The workflow runs automatically when changes are merged into `main`.

#### How it works

1. When a pull request is merged into `main`, the `Semantic Release` GitHub Actions workflow runs
2. It analyzes all commit messages since the last release to determine the next version number
3. It updates the version in `pyproject.toml` and `asu_apgap/__init__.py`, commits the change, creates a git tag, and publishes a GitHub Release

#### Commit prefix conventions

The version bump is determined by the **prefix** of the commit message:

| Prefix                                                             | Example                         | Version bump              |
| ------------------------------------------------------------------ | ------------------------------- | ------------------------- |
| `feat:`                                                            | `feat: add new API endpoint`    | **Minor** (0.1.0 → 0.2.0) |
| `fix:`                                                             | `fix: correct pagination bug`   | **Patch** (0.1.0 → 0.1.1) |
| `perf:`                                                            | `perf: optimize database query` | **Patch** (0.1.0 → 0.1.1) |
| `feat!:` or `BREAKING CHANGE:`                                     | `feat!: redesign auth flow`     | **Major** (0.1.0 → 1.0.0) |
| `build:`, `chore:`, `ci:`, `docs:`, `style:`, `refactor:`, `test:` | `chore: update dependencies`    | No version bump           |

#### Breaking changes

A breaking change can be indicated in two ways:

- Add `!` after the type: `feat!: remove deprecated endpoint`
- Add a `BREAKING CHANGE:` footer to any commit:

  ```
  feat: redesign authentication

  BREAKING CHANGE: The /auth/login endpoint now requires a JSON body instead of form data.
  ```

#### Scopes (optional)

Scopes can be added in parentheses after the prefix to provide additional context — they do not affect the version bump:

```
feat(users): add bulk user import
fix(labs): resolve membership duplication issue
```

