# CLAUDE.md — Aegis Authenticator

## What this project is

Aegis Authenticator is a free, secure, open-source 2FA (Two-Factor Authentication) app for Android. It stores TOTP/HOTP secrets in an AES-256-GCM encrypted vault with scrypt key derivation, biometric unlock, and encrypted backup support.

- **Current version:** 3.4.2 (versionCode 81)
- **License:** GNU GPL v3.0
- **Platform:** Android API 23–35, compileSdk 35

---

## Tech stack

- **Language:** Java
- **Build:** Gradle 8.10.0 with Hilt dependency injection
- **Crypto:** AES-256-GCM (AEAD), scrypt KDF (N=2^15, r=8, p=1)
- **Database:** Room (schema versioning under `app/schemas/`)
- **CI/CD:** GitHub Actions (`.github/workflows/`)
- **i18n:** Crowdin (do not submit translation PRs — handled automatically)

---

## Directory structure

```
app/
├── src/main/java/com/beemdevelopment/aegis/
│   ├── crypto/          # Encryption/decryption, vault key derivation
│   ├── database/        # Room persistence, migrations
│   ├── encoding/        # OTP encoding (Base32, Steam, Yandex)
│   ├── importers/       # Import from 10+ competing apps
│   ├── otp/             # TOTP/HOTP/Steam token generation
│   ├── services/        # Background services (Android notifications, etc.)
│   ├── ui/              # Activities, fragments, view models
│   └── util/            # General utilities
├── schemas/             # Room DB migration schemas (versioned JSON)
docs/
├── vault.md             # Complete vault file format specification
├── iconpacks.md         # Icon pack integration guide
metadata/en-US/          # F-Droid / Play Store listing assets
.github/workflows/       # build-app-workflow.yaml, CodeQL, Crowdin
```

---

## Development commands

```bash
# Build debug APK
./gradlew assembleDebug

# Build release APK (requires signing configuration)
./gradlew assembleRelease

# Run unit tests
./gradlew test

# Run instrumentation tests (requires connected device or emulator)
./gradlew connectedAndroidTest

# Clean build outputs
./gradlew clean
```

---

## Conventions

- **Package namespace:** `com.beemdevelopment.aegis.*` with logical subdomain splitting
- **DB migrations:** Room schema changes go in `app/schemas/`; never skip migration versions
- **Security rule:** No keys or sensitive material in code; all vault keys derived from user passwords at runtime
- **Translations:** Managed via Crowdin — do not merge translation PRs manually
- **No autofill:** By design; autofill slot is occupied by password managers on Android

---

## Key files

| File | Purpose |
|------|---------|
| `app/build.gradle` | Version codes, SDK levels, dependency declarations |
| `app/src/main/java/com/beemdevelopment/aegis/Preferences.java` | App settings and configuration |
| `docs/vault.md` | Vault file format and encryption specification (read before touching crypto) |
| `.github/workflows/build-app-workflow.yaml` | CI build and release pipeline |

---

## Important constraints for AI assistants

- **Security-critical code:** Changes to `crypto/` or vault parsing require careful review against the spec in `docs/vault.md`
- **No plaintext secrets:** Vault contents (OTP seeds, issuer names) are always encrypted at rest
- **Backward compatibility:** Room DB migrations must never break existing vaults — users may be upgrading from old versions
- **GPL compliance:** Any contributions must be compatible with GPLv3
