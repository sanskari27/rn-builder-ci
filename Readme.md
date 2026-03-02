# Mobile CI Engine

A **public reusable GitHub Actions workflow** for building mobile applications (Android and/or iOS) directly on GitHub Actions. Use it to **replace EAS Build**—no Expo build servers or EAS dependency. Supports React Native, Expo (via local prebuild), and native projects.

- **Manual dispatch only** — triggered via `workflow_dispatch` from your repo.
- **Split jobs** — Android on Ubuntu, iOS on macOS (macOS used only when needed).
- **Caching** — Node, Gradle, and CocoaPods caches to speed up builds.
- **Ephemeral signing** — Secrets stay in your repo; keystore/keychain are created and removed during the run.
- **Stateless & versioned** — Call by tag (e.g. `@v1`); never use `@main`.

## Quick start

1. **Add a minimal workflow** in your app repo (see [docs/usage.md](docs/usage.md) and [examples/sample-caller-workflow.yml](examples/sample-caller-workflow.yml)).
2. **Add secrets** for production builds (see [docs/secrets.md](docs/secrets.md)).
3. Run the workflow manually from the Actions tab.

That’s it. No monorepo or organization setup required.

## Documentation

| Doc | Description |
|-----|-------------|
| [Usage](docs/usage.md) | How to call the workflow, inputs, and migration from EAS Build |
| [Secrets](docs/secrets.md) | Required Android and iOS secrets for production builds |
| [Troubleshooting](docs/troubleshooting.md) | Common failures and fixes |
| [Architecture](docs/architecture.md) | Flow, caching, signing, and guarantees |


## Versioning

Use a **tag** when calling this workflow (e.g. `sanskari27/rn-builder-ci/.github/workflows/mobile-build.yml@v1.0.0`). Do **not** use `@main`. See [CHANGELOG.md](CHANGELOG.md) for release history.
