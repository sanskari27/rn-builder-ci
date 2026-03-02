# Usage

This page describes how to use the Mobile CI Engine from your app repository: add a minimal workflow, configure secrets, and run builds.

## Prerequisites

- A GitHub repository with an Android and/or iOS app (React Native, Expo, or native).
- For **production** builds: signing credentials (keystore for Android; certificate and provisioning profile for iOS) stored as repository secrets.

## 1. Add the workflow

Create a workflow file in your repo that calls the engine via `workflow_call`. For example, in `.github/workflows/build-mobile.yml`:

```yaml
name: Build Mobile

on:
  workflow_dispatch:
    inputs:
      platform:
        description: 'Platform'
        required: true
        default: 'both'
        type: choice
        options:
          - android
          - ios
          - both
      environment:
        description: 'Environment'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production

permissions:
  contents: read

jobs:
  build:
    uses: sanskari27/rn-builder-ci/.github/workflows/mobile-build.yml@v1
    with:
      platform: ${{ inputs.platform }}
      environment: ${{ inputs.environment }}
    secrets: inherit
```

If you use your own fork, replace the repo path with yours (e.g. for an individual: `YOUR_USERNAME/rn-builder-ci`). **Always use a tag** (e.g. `@v1` or `@v1.0.0`); do **not** use `@main`.

Use **`secrets: inherit`** so that your repository secrets (e.g. `IOS_CERT_BASE64`, `ANDROID_KEYSTORE_BASE64`) are forwarded to the reusable workflow. The engine does not declare a `secrets:` block—GitHub Actions does not support a "pass-through" option; the caller's `secrets: inherit` is what forwards secrets.

A full example with all optional inputs is in [../examples/sample-caller-workflow.yml](../examples/sample-caller-workflow.yml).

## 2. Add secrets (production builds)

- **Android production:** Add `ANDROID_KEYSTORE_BASE64`, `ANDROID_KEYSTORE_PASSWORD`, `ANDROID_KEY_ALIAS`, `ANDROID_KEY_PASSWORD`. See [secrets.md](secrets.md).
- **iOS (any IPA):** Add `IOS_CERT_BASE64`, `IOS_CERT_PASSWORD`, `IOS_PROFILE_BASE64`. See [secrets.md](secrets.md).

If you omit these, the workflow will fail validation (for iOS) or build with the debug keystore (Android staging only).

## 3. Run the build

1. Open the **Actions** tab in your repo.
2. Select the workflow (e.g. **Build Mobile**).
3. Click **Run workflow**, choose branch and inputs (platform, environment, etc.).
4. After the run, download artifacts from the run summary.

## Inputs (optional)

| Input | Default | Description |
|-------|--------|-------------|
| `platform` | `both` | `android`, `ios`, or `both` |
| `build_type` | (auto) | `apk`, `aab`, or `release` (release = auto-detect) |
| `environment` | `staging` | `staging` or `production` |
| `version_bump` | `false` | If true, runs `ci:version-bump` (or similar) from your `package.json` if present |
| `android_module` | `app` | Gradle module name (e.g. for multi-module projects) |
| `android_flavor` | (none) | Android product flavor |
| `ios_scheme` | (auto) | Xcode scheme |
| `ios_configuration` | (env-based) | `Debug` or `Release` |
| `ios_workspace` | (auto) | Path to `.xcworkspace` or `.xcodeproj` relative to `ios/` (e.g. `MyApp.xcworkspace`) |
| `ios_export_method` | (env-based) | IPA export: `app-store`, `ad-hoc`, `enterprise`, or `development` |
| `artifact_retention_days` | `14` | How long to keep uploaded artifacts |

**Build-time configuration:** The caller workflow's `env:` block is **not** passed into the reusable workflow (GitHub Actions does not forward job-level env to called workflows). For API URLs, feature flags, etc., use your app's normal mechanism (e.g. `.env` files, `app.config.js`, or repository variables in steps that run in the engine).

## Workflow outputs

The engine exposes two outputs you can use in a follow-up job (e.g. to post the build summary to Slack or a PR comment):

| Output | Description |
|--------|--------------|
| `summary_markdown` | Markdown summary of the run (platform, environment, status, artifact checksums, duration). |
| `failed_job` | Comma-separated list of failed job names (e.g. `validate`, `android`, `ios`), or empty if all passed. |

Example: add a second job that `needs` the build job and uses `needs.build.outputs.summary_markdown` and `needs.build.outputs.failed_job` to post results elsewhere.

## Concurrency

The engine uses `cancel-in-progress: true` scoped to the workflow and branch. **Starting a new run on the same branch cancels any in-progress run** of that workflow. Only the latest run completes. This saves minutes but can surprise teams—avoid triggering multiple builds on the same branch if you need them all to finish.

## Cross-repo and permissions

- When calling from **another repo**, the workflow runs in your repo’s context; `GITHUB_TOKEN` is your repo’s token. For a **public** engine repo, no extra token is needed.
- Use `permissions: contents: read` (or broader if needed) in the job that calls the engine.

## Migration from EAS Build

To move from EAS Build to this workflow:

1. **Export or recreate signing credentials**
   - **Android:** Export your keystore and encode it in base64; store as `ANDROID_KEYSTORE_BASE64` plus the other Android secrets.
   - **iOS:** Export the distribution certificate (`.p12`) and provisioning profile; encode in base64 and store as `IOS_CERT_BASE64` and `IOS_PROFILE_BASE64`, with `IOS_CERT_PASSWORD`.

2. **Add the workflow** (as in step 1 above) and **add the secrets** (as in step 2).

3. **Trigger builds** from the Actions tab instead of `eas build`. Artifacts (APK/AAB, IPA) are available from the workflow run.

4. You can keep using EAS for other features (updates, submit, etc.); this engine only replaces the **build** step, so you no longer depend on EAS Build or its billing for building.
