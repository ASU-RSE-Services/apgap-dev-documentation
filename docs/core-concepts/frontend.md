# APGAP Frontend

## Setup

Configuration files for the frontend are in the folder `src/environments`. There a different files for different environments (e.g. `environment.local.ts`, `environment.development.ts` and `environment.production.ts`). Each file has the following properties:

```
googleOauthClientId: '<Google OAuth Client Id>,
apiURL: '<URL of the backend server>',
googleOauthCallbackUrl: '<Callback URL for OAuth authentication>',
```

- `googleOauthClientId` will be the client id you get from Google OAuth after registering the app. 
- `apiURL` is the url of the backend server 
- `googleOauthCallbackUrl` will be the callback url for OAuth autentication (ending in `auth/login`)

For local development, you can either connect to your locally running backend (using `src/environment/environment.local.ts`) or to the backend running on the dev server (using `src/environment/environment.development.ts`). If you choose to use the dev server, you won't need to change the properties. If you choose to use your own local server, you will need to adust `googleOauthClientId`.

You can choose which environment to use when starting the development server, e.g.: 

```
ng serve --configuration=development
```

or 

```
ng serve --c development
```

If no configuration is supplied, the `local` environment is used.

## Development server

To start a local development server, run:

```bash
ng serve
```

Once the server is running, open your browser and navigate to `http://localhost:4200/`. The application will automatically reload whenever you modify any of the source files.

## Code scaffolding

Angular CLI includes powerful code scaffolding tools. To generate a new component, run:

```bash
ng generate component component-name
```

For a complete list of available schematics (such as `components`, `directives`, or `pipes`), run:

```bash
ng generate --help
```

## Building

To build the project run:

```bash
ng build
```

This will compile your project and store the build artifacts in the `dist/` directory. By default, the production build optimizes your application for performance and speed.

## Running unit tests

To execute unit tests with the [Karma](https://karma-runner.github.io) test runner, use the following command:

```bash
ng test
```

## Running end-to-end tests

For end-to-end (e2e) testing, run:

```bash
ng e2e
```

Angular CLI does not come with an end-to-end testing framework by default. You can choose one that suits your needs.

## Additional Resources

For more information on using the Angular CLI, including detailed command references, visit the [Angular CLI Overview and Command Reference](https://angular.dev/tools/cli) page.

## Development Workflow

### Semantic Versioning

This repository uses [semantic-release](https://semantic-release.gitbook.io/semantic-release/) to automatically determine and publish version numbers based on commit messages. The workflow runs automatically when changes are merged into `main`.

#### How it works

1. When a pull request is merged into `main`, the `Semantic Release` GitHub Actions workflow runs
2. It analyzes all commit messages since the last release to determine the next version number
3. It updates the version in `package.json`, generates a `CHANGELOG.md`, commits the change, creates a git tag, and publishes a GitHub Release

#### Commit prefix conventions

The version bump is determined by the **prefix** of the commit message, following the [Conventional Commits](https://www.conventionalcommits.org/) specification:

| Prefix                                                                      | Example                             | Version bump              |
| --------------------------------------------------------------------------- | ----------------------------------- | ------------------------- |
| `feat:`                                                                     | `feat: add user export feature`     | **Minor** (0.1.0 → 0.2.0) |
| `fix:`                                                                      | `fix: resolve login redirect loop`  | **Patch** (0.1.0 → 0.1.1) |
| `feat!:` or `BREAKING CHANGE:`                                              | `feat!: redesign routing structure` | **Major** (0.1.0 → 1.0.0) |
| `build:`, `chore:`, `ci:`, `docs:`, `style:`, `refactor:`, `test:`, `perf:` | `chore: update Angular to v19`      | No version bump           |

#### Breaking changes

A breaking change can be indicated in two ways:

- Add `!` after the type: `feat!: remove legacy auth flow`
- Add a `BREAKING CHANGE:` footer to any commit:

  ```
  feat: update routing module

  BREAKING CHANGE: Route paths have been restructured; existing bookmarks may be affected.
  ```

#### Scopes (optional)

Scopes can be added in parentheses after the prefix to provide additional context — they do not affect the version bump:

```
feat(auth): add Google SSO support
fix(labs): correct lab membership display
```
