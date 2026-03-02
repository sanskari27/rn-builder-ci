# Secrets

The Mobile CI Engine **does not store any secrets**. All signing credentials must be defined in the **caller repository** (your app repo) as GitHub Actions secrets and passed through when calling the workflow (e.g. with `secrets: inherit`).

## Android (production builds)

Required when `environment` is `production` and `platform` includes Android:

| Secret | Description |
|--------|-------------|
| `ANDROID_KEYSTORE_BASE64` | Your release keystore file (e.g. `.jks` or `.keystore`) encoded as base64 |
| `ANDROID_KEYSTORE_PASSWORD` | Keystore password |
| `ANDROID_KEY_ALIAS` | Key alias inside the keystore |
| `ANDROID_KEY_PASSWORD` | Key password |

**How to create base64 keystore (locally):**

```bash
base64 -i your-keystore.jks | pbcopy   # macOS
# Or on Linux: base64 -w0 your-keystore.jks
```

Paste the result into the `ANDROID_KEYSTORE_BASE64` secret value.

**Behavior:** The workflow decodes the keystore to a temporary file, uses it for the build, then deletes it. Secrets are never logged.

If these secrets are missing for a production Android build, the **Validate** job fails and the Android job does not run.

For **staging**, the workflow does not inject any signing config. The Android build uses whatever signing config is in your project's `build.gradle` (typically the debug keystore if no release signing config is defined).

---

## iOS (all IPA builds)

Required whenever `platform` includes iOS (staging or production):

| Secret | Description |
|--------|-------------|
| `IOS_CERT_BASE64` | Distribution certificate (`.p12`) encoded as base64 |
| `IOS_CERT_PASSWORD` | Password for the `.p12` file |
| `IOS_PROFILE_BASE64` | Provisioning profile (`.mobileprovision`) encoded as base64 |

**Behavior:** The workflow creates a temporary keychain, imports the cert and profile, runs the build and export, then removes the keychain and temporary files. Secrets are never logged.

If any of these secrets are missing when iOS is requested, the **Validate** job fails with a clear message and the iOS (macOS) job is not started.

---

## Staging vs production

- **Staging:** Android can run without production secrets (uses your project's default signing config, typically the debug keystore). iOS still requires the three iOS secrets for any IPA.
- **Production:** Android requires all four Android secrets; iOS requires all three iOS secrets.

No secrets are stored in the engine repo; they exist only in your repository and in the runner environment during the run.
