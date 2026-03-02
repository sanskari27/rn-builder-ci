# Troubleshooting

Common failures and how to fix them.

## Validation failures

### "Invalid platform"

- **Cause:** `platform` is not `android`, `ios`, or `both`.
- **Fix:** Pass one of those values (e.g. from your `workflow_dispatch` inputs).

### "Invalid build_type"

- **Cause:** `build_type` was set to something other than `apk`, `aab`, or `release`.
- **Fix:** Omit `build_type` for auto-detect, or use `apk` / `aab` for an explicit type. `release` also triggers auto-detect (equivalent to omitting).

### "Invalid environment"

- **Cause:** `environment` was set to something other than `staging` or `production`.
- **Fix:** Use `staging` or `production`.

### "iOS builds require signing. Add secrets: IOS_CERT_BASE64, ..."

- **Cause:** iOS was requested but one or more of `IOS_CERT_BASE64`, `IOS_CERT_PASSWORD`, or `IOS_PROFILE_BASE64` is missing.
- **Fix:** Add all three secrets in the repository that calls the workflow. See [secrets.md](secrets.md).

### "Production Android build requires secrets: ANDROID_KEYSTORE_BASE64, ..."

- **Cause:** `environment` is `production` and Android was requested, but one or more Android signing secrets are missing.
- **Fix:** Add `ANDROID_KEYSTORE_BASE64`, `ANDROID_KEYSTORE_PASSWORD`, `ANDROID_KEY_ALIAS`, and `ANDROID_KEY_PASSWORD`. See [secrets.md](secrets.md).

### "iOS requested but no ios/ folder found"

- **Cause:** `platform` includes iOS but there is no `ios/` directory (and the project is not detected as Expo).
- **Fix:** Ensure your repo has an `ios/` directory, or use an Expo project so that `expo prebuild` can generate it.

### "Android requested but no android/ folder found"

- **Cause:** `platform` includes Android but there is no `android/` directory (and the project is not detected as Expo).
- **Fix:** Ensure your repo has an `android/` directory, or use an Expo project so that `expo prebuild` can generate it.

---

## Build failures

### Android: Gradle or build fails

- **Keystore / signing:** Ensure `ANDROID_KEYSTORE_BASE64` is the full base64 of the keystore file (no extra newlines). Alias and passwords must match the keystore.
- **Module/flavor:** If your app is not the default `:app` module or uses product flavors, set `android_module` and optionally `android_flavor` in the workflow inputs. The flavor name is automatically capitalized for the Gradle task (e.g. `staging` → `assembleStagingRelease`).
- **Staging signing:** Staging Android builds use whatever signing config is in your project's `build.gradle`. If no release signing config is defined, Gradle uses the debug keystore by default. The workflow does **not** inject signing for staging—only production.
- **Cache:** The first run (cold cache) is slower; subsequent runs use the Gradle cache. If the cache key changed (e.g. after Gradle or config changes), the next run may be slow again.

### iOS: Xcode or export fails

- **Signing:** Ensure the provisioning profile matches the certificate and your app’s bundle ID. The profile must be installed (the workflow imports the one you provide).
- **Scheme:** If you have multiple schemes, set `ios_scheme` to the one that builds the app.
- **Workspace:** If you have multiple `.xcworkspace` or `.xcodeproj` under `ios/`, set `ios_workspace` to the path relative to `ios/` (e.g. `MyApp.xcworkspace`). If you pass `ios/MyApp.xcworkspace`, the `ios/` prefix is stripped automatically.
- **Export / IPA:** The workflow generates `ExportOptions.plist` from your provisioning profile (method + bundle ID → profile name). If export still fails, set `ios_export_method` explicitly (`app-store`, `ad-hoc`, `enterprise`, or `development`) to match your profile type. For apps with extensions (multiple bundle IDs), the generated plist may need manual provisioning profile entries—document your export requirements or use a repo-owned `ExportOptions.plist` if the engine adds support for it.

### "version_bump requested but no version-bump script found"

- **Cause:** The workflow was called with `version_bump: true` but your `package.json` has no script named `ci:version-bump` (or `version-bump`).
- **Fix:** Either add a script (e.g. `"ci:version-bump": "node scripts/bump-version.js"`) or set `version_bump: false`. The workflow continues without failing; it only logs a warning.

---

## Cache and performance

- **Cache key changes:** Node cache is keyed by lockfile hash; Gradle by Gradle files; CocoaPods by `Podfile.lock`. Changing dependencies or config invalidates the cache and the next run will be slower.
- **No lockfile:** If there is no `package-lock.json`, `yarn.lock`, or `pnpm-lock.yaml`, the Node cache is not used and a warning is logged.
- **pnpm:** If `pnpm-lock.yaml` is detected, the workflow automatically installs pnpm via `pnpm/action-setup@v4`. The pnpm version is read from the `packageManager` field in `package.json` if available.
- **CocoaPods version:** If a `Gemfile` is present (at the repo root or in `ios/`), the workflow runs `bundle install && bundle exec pod install` to use the version pinned in your Gemfile. Otherwise, it uses the system-installed CocoaPods.

---

## Artifacts and summary

- Artifacts appear in the **Actions** run summary under the **Artifacts** section. Download from there.
- Retention is controlled by `artifact_retention_days` (default 14).
- The build summary job outputs a short markdown summary (platform, environment, commit, status, and artifact checksums when available). Use the workflow outputs `summary_markdown` and `failed_job` if you need them in a downstream job.
