# Ambulance App Config Guide

This guide explains the Android app config, app update workflow, Notices, and how document/helper versions trigger remote refreshes.

The static files referenced by app config live in:

```text
https://github.com/yniwashi/pdf-viewer
https://docs.niwashibase.com
```

## Live App Config

The live app config Gist is on the `yazan414` account:

```text
https://gist.github.com/yazan414/2ed2d30193b3dedffcf789981ad14c0e
```

Raw URL used by Android production builds:

```text
https://gist.githubusercontent.com/yazan414/2ed2d30193b3dedffcf789981ad14c0e/raw/ambulance_app_config.json
```

Testing Gist:

```text
https://gist.github.com/yazan414/327274b93c586ce8b18900c38982b3cd
```

Testing raw URL:

```text
https://gist.githubusercontent.com/yazan414/327274b93c586ce8b18900c38982b3cd/raw/ambulance_app_config_testing.json
```

## What The App Config Controls

Android version `2.1+` checks one config file:

```text
ambulance_app_config.json
```

It controls:

- app update dialog
- forced update behavior
- app download URL
- in-app Notice popup
- Notice bell history
- config check interval
- CPG/SOP/CPM/PAT PDF and index URLs/versions
- RSI checklist HTML URL/version/image visibility
- CCP/AP pediatric dosing helper URLs/versions
- Websites helper URL/version

Version `2.0` users still use older update logic already compiled into that APK.

## Related Guides

Docs/PDF guide:

```text
https://github.com/yniwashi/pdf-viewer/blob/main/docs/README.md
```

Helpers guide:

```text
https://github.com/yniwashi/pdf-viewer/blob/main/helpers/README.md
```

Viewer guide:

```text
https://github.com/yniwashi/pdf-viewer/blob/main/viewer/README.md
```

Docs host root guide:

```text
https://github.com/yniwashi/pdf-viewer/blob/main/README.md
```

## Check Interval

This field controls how often Android tries to fetch app config:

```json
"config_check_interval_hours": 3
```

If GitHub/Gist is unreachable, Android uses:

1. in-memory config if already loaded
2. saved cached config from the last successful fetch
3. bundled fallback config inside the APK

The app does not replace saved config unless the remote JSON downloads and parses successfully.

## Switching Between Production And Testing

In `AppConfigRepository.kt`, switch `APP_CONFIG_URL`.

Production:

```kotlin
private const val APP_CONFIG_URL =
    "https://gist.githubusercontent.com/yazan414/2ed2d30193b3dedffcf789981ad14c0e/raw/ambulance_app_config.json"
```

Testing:

```kotlin
private const val APP_CONFIG_URL =
    "https://gist.githubusercontent.com/yazan414/327274b93c586ce8b18900c38982b3cd/raw/ambulance_app_config_testing.json"
```

If the testing URL returns `404`, confirm the Gist file name is exactly:

```text
ambulance_app_config_testing.json
```

## Bundled Fallback

Android also ships a fallback config inside the APK:

```text
app/src/main/assets/ambulance_app_config.json
```

Fallback order:

1. Use memory config if already loaded and still inside the check interval.
2. Use saved cached config from the last successful remote fetch.
3. Try fetching the remote Gist if the check interval has passed.
4. If remote fetch fails, keep using saved cached config.
5. If there is no saved cache, use bundled fallback from APK assets.

Important:

- Clearing app data removes saved cached config.
- Uninstalling removes saved cached config.
- Reinstalling brings back the bundled fallback because it is part of the APK.
- The Gist should be the live source of truth.
- Keep bundled fallback reasonable before every APK release.

## App Update Section

Example:

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

Fields:

- `latest_version`: newest available app version.
- `min_supported_version`: oldest app version allowed to keep using the app.
- `release_title`: title shown in the update dialog.
- `release_message`: update message.
- `download_url`: opened by the Download button.
- `force_update`: if `true`, the app hides main buttons until update.

The local installed app version comes from `BuildConfig.VERSION_NAME`.

Do not reintroduce a hardcoded current version in `MainActivity`.

## Releasing A New App Version

Example: release `2.2`.

1. Update Android Gradle version:

```gradle
versionCode 7
versionName "2.2"
```

2. Build and upload the APK.
3. Update Gist `app_update.latest_version`.
4. Update `min_supported_version` if older versions should be blocked.
5. Update `release_title`, `release_message`, and `download_url`.
6. Set `force_update`.
7. Save the Gist.
8. Test on a fresh install and an existing install.

Installed apps see the update after `config_check_interval_hours` passes, or sooner if they have no cached config.

## Notice Announcement

The app can show an in-app Notice popup without sending a push notification.

Example:

```json
"announcement": {
  "enabled": true,
  "id": "notice_2026_05_10",
  "title": "New CPG update",
  "message": "CPG has been updated. Open Guidelines to view the latest version.",
  "button_text": "Got it",
  "date": "May 10, 2026"
}
```

Fields:

- `enabled`: set `true` to activate.
- `id`: unique ID used for dismissal tracking.
- `title`: Notice title.
- `message`: Notice body.
- `button_text`: dismiss button text.
- `date`: publish date shown in the bell inbox.

Change `announcement.id` when you want the Notice to auto-popup again for users who dismissed the old Notice.

Use `\n` for line breaks:

```json
"message": "Line one.\nLine two."
```

## Notice Bell History

The bell inbox uses the top-level `notices` array.

Example:

```json
"notices": [
  {
    "enabled": true,
    "id": "notice_2026_05_10",
    "title": "New CPG update",
    "message": "CPG has been updated. Open Guidelines to view the latest version.",
    "button_text": "OK",
    "date": "May 10, 2026"
  }
]
```

Rules:

- Every Notice needs a stable unique `id`.
- `enabled: true` shows it in the inbox.
- Read/unread state is stored locally on the device.
- The bell badge is not Firebase notification history.
- If `notices` is missing or empty, the app can fall back to the active `announcement`.
- The app removes duplicates by `id`.

Recommended setup:

1. Put the newest important Notice in `announcement` if it should auto-popup.
2. Add the same Notice to `notices` so it remains visible in the bell history.
3. Keep older Notices in `notices` as long as users should review them.

## Firebase Console Push Notifications

Firebase Console notifications are reminders only. They are not the reliable source for the bell inbox.

Recommended Firebase message:

```text
New Ambulance notice available. Open the app and tap the bell.
```

Why:

- Firebase Console sends notification messages.
- When the device is locked, backgrounded, or the app is killed, Android/Firebase may display the notification directly.
- In that situation, the app may not receive the notification content.
- If the app does not receive it, it cannot add it to the in-app inbox.

Reliable Notice workflow:

1. Add the Notice to `notices` in app config.
2. Optionally set it as `announcement`.
3. Send Firebase push only to tell users to open the app/check the bell.

Opening MainActivity clears visible non-CPR app notifications when possible.

## Documents Array

Documents live in the top-level `documents` array.

Example:

```json
{
  "type": "CPG",
  "version": "2.5",
  "pdf_url": "https://docs.niwashibase.com/docs/cpg-81w9d1f.pdf",
  "index_url": "https://docs.niwashibase.com/helpers/cpg_index.json"
}
```

Current document types:

```text
CPG
SOP
CPM
PAT
```

Current PDFs:

```text
https://docs.niwashibase.com/docs/cpg-81w9d1f.pdf
https://docs.niwashibase.com/docs/sop-101qq9f2w.pdf
https://docs.niwashibase.com/docs/cpm-202e9d33q.pdf
https://docs.niwashibase.com/docs/pat-301h6j54r.pdf
```

Current indexes:

```text
https://docs.niwashibase.com/helpers/cpg_index.json
https://docs.niwashibase.com/helpers/sop_index.json
https://docs.niwashibase.com/helpers/cpm_index.json
https://docs.niwashibase.com/helpers/pat_index.json
```

To update a document:

1. Update the PDF and/or index helper in `yniwashi/pdf-viewer`.
2. Update `pdf_url` or `index_url` in app config if the filename changed.
3. Increase the document `version`.
4. Save the Gist.
5. Test Android Guidelines/Search.

Android refreshes cached PDFs and indexes when the document `version` changes.

Full document update guide:

```text
https://github.com/yniwashi/pdf-viewer/blob/main/docs/README.md
```

## RSI Checklist Entry

RSI is configured in the `documents` array:

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

1. Update `helpers/rsi_checklist_js_android.html`.
2. Increase RSI `version`.
3. Set `show_image`.
4. Save app config.
5. Test Android RSI.

Android processes the RSI HTML by replacing Android placeholders with bundled local image/audio paths before displaying it.

## Pediatric Dosing Helpers

Pediatric dosing helpers are listed under:

```json
"pediatric_dosing": {
  "enabled": true,
  "helpers": []
}
```

Current entries:

```json
{
  "id": "ccp_pediatric_dosing",
  "scope": "CCP",
  "age_groups": ["months", "years"],
  "schema_version": "0.1",
  "version": "0.1",
  "url": "https://docs.niwashibase.com/helpers/ccp_pediatric_dosing_helper.json",
  "fallback_asset": "ccp_pediatric_dosing_helper.json",
  "enabled": true
}
```

```json
{
  "id": "ap_pediatric_dosing",
  "scope": "AP",
  "age_groups": ["months", "years"],
  "schema_version": "0.1",
  "version": "0.1",
  "url": "https://docs.niwashibase.com/helpers/ap_pediatric_dosing_helper.json",
  "fallback_asset": "ap_pediatric_dosing_helper.json",
  "enabled": true
}
```

Android validates:

- helper `helper_type`
- helper `schema_version`
- helper `version`
- non-empty `medications`
- at least one renderable medication button

Remote update rule:

- Change helper `version` for medication content changes.
- Keep helper top-level `version` synchronized with app-config helper `version`.
- Change `schema_version` only when the JSON contract changes and Android supports that change.
- Keep bundled fallback assets updated before release builds.

Full helper editing guide:

```text
https://github.com/yniwashi/pdf-viewer/blob/main/helpers/README.md
```

## Websites Helper

Websites are configured under:

```json
"websites": {
  "enabled": true,
  "schema_version": "0.1",
  "version": "0.1",
  "url": "https://docs.niwashibase.com/helpers/websites.json",
  "fallback_asset": "websites.json"
}
```

The helper file lives at:

```text
https://docs.niwashibase.com/helpers/websites.json
```

The helper should include matching top-level metadata:

```json
{
  "helper_type": "websites",
  "schema_version": "0.1",
  "version": "0.1",
  "websites": []
}
```

Android should validate:

- `helper_type` is `websites`
- helper `schema_version` matches app config
- helper `version` matches app config
- `websites` contains at least one enabled usable item
- each usable item has a non-blank `title` and `url`

Remote update rule:

- Change `websites.version` for website additions/removals, title/category/subtitle changes, URL changes, enabled/disabled changes, order changes, and icon URL changes.
- Keep helper top-level `version` synchronized with app-config `websites.version`.
- Change `schema_version` only when the websites JSON contract changes and Android supports that change.
- Keep bundled fallback `assets/websites.json` updated before release builds.
- Website icons can be remote through `icon_url`; Android should cache icons and fall back gracefully if an icon fails.

## Safe Rules

- Do not rename `schema_version`.
- Do not rename `config_check_interval_hours`.
- Do not rename `app_update`.
- Do not rename `announcement`.
- Do not rename `notices`.
- Do not rename `documents`.
- Do not rename `pediatric_dosing`.
- Do not rename `websites` after Android starts using it.
- You can add new fields later. The app should ignore unknown fields.
- Increase a version when you want Android to refresh that item.
- Keep version values simple, for example `2.1`, `2.2`, `"4.1"`.
- For Notice popup replay, change `announcement.id`.
- For bell inbox Notices, each Notice needs a stable unique `id`.
- For CCP/AP pediatric dosing, helper `schema_version` and `version` must match app config.
- For Websites, helper `schema_version` and `version` must match app config.

## Quick Update Checklist

For app release:

1. Build APK.
2. Upload APK.
3. Update `app_update`.
4. Test update dialog.

For document update:

1. Update PDF/index in `pdf-viewer`.
2. Increase document `version` in app config.
3. Test Guidelines/Search.

For helper update:

1. Update helper in `pdf-viewer/helpers`.
2. Increase matching helper/document version in app config.
3. Update Android bundled fallback if applicable.
4. Test the app feature.

For Notice:

1. Add to `notices`.
2. Add to `announcement` if popup is needed.
3. Use a new `id` if users should see it again.
4. Optionally send Firebase push as a reminder only.
