# iOS build with a free Apple ID (optional CI signing)

> **Status:** The GitHub workflow for this path is **disabled by default**. It runs the real macOS build only when the repository variable `ENABLE_IOS_SIGNED_CI` is set to `true` and the signing secrets below are configured.
>
> **Recommended for most users (no secrets, no Mac session):** [unsigned IPA + Sideloadly on Windows](ios-sideload-install.md) via the **Build unsigned iOS IPA** workflow.

Hermes Mobile is a **Tauri v2** app. Apple only allows iOS archives on **macOS with Xcode**. If you do not own a Mac, you can use **GitHub Actions** (`macos-latest`) as a cloud Mac and install the resulting `.ipa` with **AltStore** or **Sideloadly** on your iPhone.

This document describes an **optional** flow where CI signs the `.ipa` using certificates you export once on a Mac and store as GitHub **secrets**—useful if you want a pre-signed artifact without re-signing every build on Windows.

A **paid** Apple Developer account is **not** required for personal sideloading. You still need a **one-time** Mac session (borrowed machine, friend, internet café, etc.) to export signing material for CI.

## Enable the workflow

1. Fork the repository (or use your own copy).
2. **Settings → Secrets and variables → Actions → Variables** → New repository variable: `ENABLE_IOS_SIGNED_CI` = `true`.
3. Add the secrets listed below.
4. **Actions → iOS IPA (free Apple ID, CI signing) → Run workflow**.

Until `ENABLE_IOS_SIGNED_CI` is `true`, manual runs only show a short “disabled” job and do not consume macOS minutes.

## What you end up with

1. Push signing secrets to your GitHub fork.
2. Set `ENABLE_IOS_SIGNED_CI=true`.
3. Run **Actions → iOS IPA (free Apple ID, CI signing) → Run workflow**.
4. Download the **hermes-mobile-ios-ipa** artifact.
5. Install on iPhone via **AltStore** or **Sideloadly** (Windows + AltServer is fine; no Mac needed day to day).

Free AltStore installs expire after **about 7 days** and must be refreshed. Free tier limits how many apps can be active at once.

## One-time setup on any Mac (free Apple ID)

### 1. Prerequisites on the Mac

- Install **Xcode** from the App Store and open it once to accept the license.
- Install **Node 22+**, **pnpm**, and **Rust** (`rustup target add aarch64-apple-ios`).
- Clone this repo and run `pnpm install`.

### 2. Sign in and register your iPhone

1. **Xcode → Settings → Accounts** → add your **free Apple ID**.
2. Connect your iPhone with USB, unlock it, and tap **Trust**.
3. From the repo root:

   ```bash
   pnpm exec tauri ios init
   pnpm exec tauri ios dev
   ```

   Pick your **physical device** when prompted. Xcode may ask to enable **Developer Mode** on the phone (Settings → Privacy & Security → Developer Mode).

The first successful device run creates an **Apple Development** certificate and a provisioning profile for bundle ID **`com.hermesagent.mobile`**.

### 3. Find your Personal Team ID

In Xcode → **Settings → Accounts** → select your Apple ID → **Manage Teams…**  
Copy the **Team ID** (10 characters, e.g. `ABCDE12345`). This is `APPLE_DEVELOPMENT_TEAM` (not your email).

### 4. Export the certificate (`.p12`)

1. Open **Keychain Access** on the Mac.
2. Find **Apple Development: your@email.com** under **My Certificates** (expand to see the private key).
3. Right-click the certificate → **Export** → save as `HermesMobileDev.p12` and choose a password (remember it).

Base64 for GitHub (macOS):

```bash
base64 -i HermesMobileDev.p12 | pbcopy
```

Paste into secret **`IOS_CERTIFICATE`**. Put the export password in **`IOS_CERTIFICATE_PASSWORD`**.

### 5. Export the provisioning profile

After a successful `tauri ios dev` on your device, Xcode stores profiles under:

`~/Library/MobileDevice/Provisioning Profiles/`

Find the newest `.mobileprovision` that includes **`com.hermesagent.mobile`** and your device (open in a text editor and search for `com.hermesagent.mobile`).

Base64:

```bash
base64 -i /path/to/profile.mobileprovision | pbcopy
```

Paste into secret **`IOS_MOBILE_PROVISION`**.

> **Tip:** If you change phones or the profile expires, repeat `tauri ios dev` on the Mac and re-export the profile (certificate can usually stay the same).

## GitHub repository secrets

In your fork: **Settings → Secrets and variables → Actions → New repository secret**

| Secret | Value |
|--------|--------|
| `APPLE_DEVELOPMENT_TEAM` | 10-character Personal Team ID |
| `IOS_CERTIFICATE` | Base64 of the `.p12` (single line, no extra whitespace) |
| `IOS_CERTIFICATE_PASSWORD` | Password used when exporting `.p12` |
| `IOS_MOBILE_PROVISION` | Base64 of the `.mobileprovision` file |

Never commit these values to git.

## Run the workflow

**Actions** tab → **iOS IPA (free Apple ID, CI signing)** → **Run workflow** → wait for the green check → open the run → **Artifacts** → download **hermes-mobile-ios-ipa**.

The workflow uses Tauri’s **`debugging`** export method and [manual signing env vars](https://v2.tauri.app/distribute/sign/ios/), which match a free **Apple Development** certificate.

## Install with AltStore (Windows is OK)

1. Install [iTunes](https://www.apple.com/itunes/) or Apple Devices app and [AltServer for Windows](https://altstore.io/).
2. Install **AltStore** on the iPhone (same Apple ID as on the Mac during setup is simplest).
3. AirDrop / email / cloud the `.ipa` to the phone, or use AltStore’s **Install from file**.
4. Refresh before the 7-day window ends (keep AltServer running on your PC on the same network).

**Sideloadly** follows the same idea: pick the `.ipa`, sign with your Apple ID, install over USB or Wi‑Fi.

### If install fails

- **Bundle ID conflict:** Another sideloaded app may already use `com.hermesagent.mobile`. Remove the old copy or change `identifier` in `src-tauri/tauri.conf.json` on a fork (then rebuild).
- **Expired signature:** Re-run the GitHub workflow or refresh in AltStore.
- **Profile missing device:** Re-run `tauri ios dev` with the new phone connected and update `IOS_MOBILE_PROVISION`.

## What a free Apple ID cannot do

- **TestFlight** or **App Store** release (paid program).
- **App Store Connect API** automatic signing in CI (paid program).
- Long-lived installs without periodic refresh (7-day limit on free personal certificates / AltStore).

## Related docs

- Primary sideload path: [ios-sideload-install.md](ios-sideload-install.md)
- [Tauri iOS code signing](https://v2.tauri.app/distribute/sign/ios/)
- Workflow: [`.github/workflows/ios-ipa-free-apple-id.yml`](../.github/workflows/ios-ipa-free-apple-id.yml)