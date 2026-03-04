# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

- Initial implementation of the reusable mobile build workflow.

## [1.0.0] - TBD

### Added

- Reusable workflow `mobile-build.yml` (trigger: `workflow_call` only).
- **Validate** job (Ubuntu): input validation, project detection (React Native / Expo / native), and secret checks; fails early so macOS is not used when iOS is misconfigured.
- **Android** job (Ubuntu): checkout, Node and Gradle caching, optional version bump, Expo prebuild when applicable, Java 17, ephemeral keystore for production signing, build (APK/AAB), checksum, artifact upload.
- **iOS** job (macOS): checkout, Node and CocoaPods caching, optional version bump, Expo prebuild when applicable, ephemeral keychain and signing, Xcode build and export, checksum, artifact upload.
- **Summary** job (Ubuntu): structured build summary (platform, environment, commit, status, artifact checksums) and optional job summary output.
- Inputs: `platform`, `build_type`, `environment` only (3 inputs for initial React Native setup). Android module `app`, iOS scheme/workspace/export auto from `ios/`, artifact retention 14 days, version bump skipped.
- Concurrency: cancel in-progress runs on the same branch.
- Documentation: usage, secrets, troubleshooting, architecture.
- Example caller workflow and migration guidance from EAS Build.
