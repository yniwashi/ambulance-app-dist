# Ambulance App Config Guide

This folder has the reference copy of `ambulance_app_config.json`.

The live app config Gist is on yazan414 account:

```text
https://gist.github.com/yazan414/2ed2d30193b3dedffcf789981ad14c0e
```

The raw URL used by the Android app is:

```text
https://gist.githubusercontent.com/yazan414/2ed2d30193b3dedffcf789981ad14c0e/raw/ambulance_app_config.json
```

The testing app config Gist is:

```text
https://gist.github.com/yazan414/327274b93c586ce8b18900c38982b3cd
```

The testing raw URL is:

```text
https://gist.githubusercontent.com/yazan414/327274b93c586ce8b18900c38982b3cd/raw/ambulance_app_config_testing.json
```

## What The App Checks

Version `2.1+` checks only one file:

```text
ambulance_app_config.json
```

That one file controls:

- app update messages
- app download URL
- how often the app checks the config
- CPG/SOP/CPM/PAT PDF and index versions
- RSI checklist HTML version and image visibility

Version `2.0` users still use the old update Gist because that code is already inside their installed APK.

## Check Interval

This controls how often the app tries to fetch the config:

```json
"config_check_interval_hours": 3
```

If GitHub is unreachable, the app uses:

1. cached config from the last successful fetch
2. bundled fallback config inside the APK

The app does not replace the saved config unless the remote JSON downloads and parses successfully.

## Testing Gist

Use the testing Gist when you want to test the update alert without changing the production app config.

In `AppConfigRepository.kt`, switch `APP_CONFIG_URL`:

```kotlin
// Production app config Gist. Keep this active for normal releases.
private const val APP_CONFIG_URL =
    "https://gist.githubusercontent.com/yazan414/2ed2d30193b3dedffcf789981ad14c0e/raw/ambulance_app_config.json"

// Testing app config Gist for checking the update alert before changing production.
// private const val APP_CONFIG_URL =
//     "https://gist.githubusercontent.com/yazan414/327274b93c586ce8b18900c38982b3cd/raw/ambulance_app_config_testing.json"
```

To test:

1. Comment the production `APP_CONFIG_URL`.
2. Uncomment the testing `APP_CONFIG_URL`.
3. Make the testing Gist `app_update.latest_version` higher than local `versionName`.
4. Install/run the app.

If the testing URL returns `404`, check that the file name inside the Gist is exactly:

```text
ambulance_app_config_testing.json
```

If the Gist has only one file, a filename-less raw URL can also work:

```text
https://gist.githubusercontent.com/yazan414/327274b93c586ce8b18900c38982b3cd/raw
```

## Bundled Fallback

The app also ships with a local fallback config inside the APK:

```text
app/src/main/assets/ambulance_app_config.json
```

This file is copied into the APK at build time. It is not downloaded from GitHub.

Fallback order:

1. Use memory config if already loaded and still inside the check interval.
2. Use saved cached config from the last successful remote fetch.
3. Try fetching the remote Gist if the check interval has passed.
4. If remote fetch fails, keep using saved cached config.
5. If there is no saved cache, use bundled fallback from APK assets.

Important:

- Clearing app data removes the saved cached config.
- Uninstalling removes the saved cached config.
- Reinstalling brings back the bundled fallback because it is part of the APK.
- If the remote Gist URL is wrong and there is no saved cache, the app uses the bundled fallback.

The current bundled fallback has:

```json
"app_update": {
  "latest_version": 2.0,
  "min_supported_version": 2.0
}
```

So if your local app `versionName` is also `2.0`, no update alert appears when the app falls back to the bundled config.

When preparing a real release, keep the bundled fallback reasonable but do not rely on it for live updates. The Gist should be the live source of truth.

## App Update Section

```json
"app_update": {
  "latest_version": 2.0,
  "min_supported_version": 2.0,
  "release_title": "CPG & Other Updates",
  "release_message": "Message shown to the user.",
  "download_url": "https://update.niwashibase.com/apk",
  "force_update": true
}
```

Fields:

- `latest_version`: newest app version available.
- `min_supported_version`: oldest version allowed to keep using the app.
- `release_title`: title shown in the update dialog.
- `release_message`: main update message.
- `download_url`: where the Download button opens.
- `force_update`: if `true`, the app hides main buttons until the user updates.

## Releasing A New App Version

Example: releasing version `2.2`.

1. Update `app/build.gradle`:

```gradle
versionCode 7
versionName "2.2"
```

2. Build and upload the APK.

3. Update the Gist config:

```json
"app_update": {
  "latest_version": 2.2,
  "min_supported_version": 2.1,
  "release_title": "Ambulance App v2.2",
  "release_message": "Write the update details here.",
  "download_url": "https://update.niwashibase.com/apk",
  "force_update": true
}
```

4. Save the Gist.

Installed apps will see the update after `config_check_interval_hours` passes, or sooner if the app has no cached config.

## Updating Documents

Documents live in the `documents` array.

Example:

```json
{
  "type": "CPG",
  "version": "2.5",
  "pdf_url": "https://docs.niwashibase.com/docs/cpg-81w9d1f.pdf",
  "index_url": "https://docs.niwashibase.com/helpers/cpg_index.json"
}
```

To update a document:

1. Upload the new PDF or helper index JSON.
2. Update `pdf_url` or `index_url` if the URL changed.
3. Increase `version`.
4. Save the Gist.

The app refreshes cached PDFs and indexes when the document `version` changes.

## RSI Checklist

RSI is also inside `documents`:

```json
{
  "id": "rsi_checklist",
  "type": "html",
  "version": "4.0",
  "url": "https://docs.niwashibase.com/helpers/rsi_checklist_js_android.html",
  "show_image": true
}
```

To update RSI:

1. Upload the new RSI HTML.
2. Increase RSI `version`, for example from `"4.0"` to `"4.1"`.
3. Set `show_image` to `true` or `false`.
4. Save the Gist.

The Android app processes the RSI HTML and replaces the image/audio placeholders with local app files.

## Safe Rules

- Do not rename `schema_version`.
- Do not rename `config_check_interval_hours`.
- Do not rename `app_update`.
- Do not rename `documents`.
- You can add new fields later. The app ignores fields it does not know.
- Increase a version when you want the app to refresh that item.
- Keep version numbers simple, for example `2.1`, `2.2`, `"4.1"`.
