# Install Hermes Mobile on iPhone (unsigned IPA + Sideloadly)

Hermes Mobile is not distributed on the App Store. This path builds an **unsigned** `.ipa` in GitHub Actions (macOS runner only—you do not need your own Mac) and lets **you** sign the app with your personal Apple ID on Windows or macOS.

No Apple distribution certificates or repository secrets are required for the build.

## 1. Get the unsigned IPA

1. Open the repository on GitHub → **Actions** → **Build unsigned iOS IPA**.
2. Click **Run workflow** (branch `main`) and wait for the job to finish (first run may take a while).
3. Under **Artifacts**, download **hermes-mobile-ios-unsigned** (`Hermes-Mobile-unsigned.ipa`).

Pushes to **`main`** that touch the app or this workflow also trigger a build automatically. Tagged releases (`v*`) do too. You can always start one manually from **Actions → Run workflow**.

**macOS runner queues:** GitHub sometimes cannot assign a hosted Mac runner for about 15 minutes and cancels the job with *The job was not acquired by Runner*. That is a capacity issue, not a problem with your commit. Use **Re-run all jobs** on the failed run, or wait—after this workflow lands on `main`, **Retry unsigned iOS IPA (runner capacity)** re-runs the build automatically (up to five attempts).

## 2. Sign and install (Windows)

1. Install [iTunes](https://www.apple.com/itunes/download/win64/) (or Apple Devices) so Apple mobile drivers are available.
2. Install [Sideloadly](https://sideloadly.io/).
3. Connect the iPhone with USB, unlock it, and trust the PC.
4. In Sideloadly: select the device, drag in `Hermes-Mobile-unsigned.ipa`, enter your **Apple ID** (app-specific password if prompted).
5. After install: on the iPhone go to **Settings → General → VPN & Device Management** and trust the developer profile.

**Free Apple ID limits:** sideload certificates often expire after about **7 days**; re-sign in Sideloadly when the app will not open. Free accounts also limit how many sideloaded apps you can keep at once.

## 3. Pair Hermes on the phone

Follow the in-app **Security & Pairing** flow: Tailscale on host and phone, your PC’s Tailscale gateway URL (not `127.0.0.1`), then sign in to **your** Hermes gateway.

Official gateway guidance: [Hermes multi-connection desktop](https://hermes-agent.nousresearch.com/docs/user-guide/multi-connection-desktop).

## Trust model

- CI does **not** sign with a publisher certificate you must trust.
- Sideloadly replaces signing with **your** Apple ID.
- You can fork the repo, run the workflow on **your** commit, and compare artifacts before signing.

## AltStore (optional)

You can use [AltStore](https://altstore.io/) on device instead of Sideloadly on Windows; the same unsigned IPA applies. AltStore can refresh sideloads on a schedule while on the same Wi‑Fi as AltServer.

## Optional: CI-signed IPA (disabled by default)

If you later configure Apple Development secrets in GitHub and set `ENABLE_IOS_SIGNED_CI=true`, you can use the **iOS IPA (free Apple ID, CI signing)** workflow instead. See [ios-free-apple-id.md](ios-free-apple-id.md).