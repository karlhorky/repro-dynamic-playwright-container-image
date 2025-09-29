# Repro for Dynamic Playwright Container Image in GitHub Actions

Reproduction of reading the Playwright version in `package.json` and using it to set a dynamic version of the Playwright Docker container image in GitHub Actions:

- Playwright version set in `package.json` (`@playwright/test` version in `devDependencies`)
- GitHub Actions workflow [`.github/workflows/ci-container.yml`](https://github.com/karlhorky/repro-dynamic-playwright-container-image/blob/main/.github/workflows/ci-container.yml) which reads this version and runs tests in a container with two jobs:
  - `resolve-playwright-version`: Read exact version of `@playwright/test` from `package.json` and set it as output for the `ci` job
  - `ci`: Uses the resolved Playwright version in the `jobs.<job_id>.container.image` expression to run tests in a container using an image version matching the Playwright version

If you don't use an exact version of `@playwright/test` in `package.json`, then try either of these approaches:

1. Retrieving the version from your package manager's lockfile (`package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `bun.lock`, etc.)
2. Installing all packages and retrieving the version from `node_modules/@playwright/test/package.json`

## Why

Playwright docs mention two ways of installing Playwright browsers in CI:

1. [`playwright install --with-deps`](https://playwright.dev/docs/ci#on-pushpull_request)
2. [Via Docker container with `microsoft/playwright` image](https://playwright.dev/docs/ci#via-containers)

Installing Playwright browsers with [`playwright install --with-deps`](https://playwright.dev/docs/ci#on-pushpull_request) can lead to long installation times (can be multiple minutes to install dependencies):

- Install dependencies: 1m02s
- Total: 1m17s

<img width="1445" height="814" alt="Screenshot 2025-09-29 at 14 45 18" src="https://github.com/user-attachments/assets/ef8676eb-bd89-43cc-ba74-be7517200b2d" />

Running Playwright [via container](https://playwright.dev/docs/ci#via-containers) (using [the official Microsoft Docker image](https://hub.docker.com/r/microsoft/playwright)) is faster:

- Initialize Docker container: 25s
- Total: 53s

<img width="1437" height="686" alt="Screenshot 2025-09-29 at 14 45 47" src="https://github.com/user-attachments/assets/1c2a5372-8152-4a8e-8c90-6c738509492f" />

<img width="1438" height="851" alt="Screenshot 2025-09-29 at 14 46 09" src="https://github.com/user-attachments/assets/398b3e00-b86b-480d-820d-a78015f44179" />

However, the Docker container approach hardcodes the Playwright version in another place in the codebase - the GitHub Actions workflow files - requiring effort or automation to keep the Playwright version in `package.json` and the Docker image version in sync (high chance of getting out of sync as Playwright is upgraded).

This reproduction treats the version in `package.json` as the single source of truth and uses it to dynamically set the Docker image version in the GitHub Actions workflow.
