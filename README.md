# Ambulance App Config Guide

This folder has the reference copy of `ambulance_app_config.json`.

The live app config Gist is:

```text
https://gist.github.com/yazan414/2ed2d30193b3dedffcf789981ad14c0e
```

The raw URL used by the Android app is:

```text
https://gist.githubusercontent.com/yazan414/2ed2d30193b3dedffcf789981ad14c0e/raw/ambulance_app_config.json
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
