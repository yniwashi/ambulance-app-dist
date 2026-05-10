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
- in-app Notice announcements
- Notice bell history
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

## Notice Announcement Section

The app can show a centered in-app Notice message without sending a push notification.
This is useful for non-urgent messages that should appear when the user opens the app.

The app also has a bell icon in the MainActivity title area. The bell opens the Notice inbox/history.
The bell inbox is driven by the app config, not by Firebase Console notification history.

Add this top-level object beside `app_update` and `documents`:

```json
"announcement": {
  "enabled": true,
  "id": "2026-05-05-001",
  "title": "Notice",
  "message": "CPG, SOP, CPM, and PAT documents have been updated.",
  "button_text": "Got it!",
  "date": "May 10, 2026"
}
```

Fields:

- `enabled`: set to `true` to activate the Notice, or `false` to hide it.
- `id`: unique ID for this message. It can be any text, not only numbers.
- `title`: title shown at the top of the Notice dialog.
- `message`: the main message shown to the user.
- `button_text`: text on the dismiss button.
- `date`: publish date shown in the bell inbox.

Simple disabled example:

```json
"announcement": {
  "enabled": false,
  "id": "",
  "title": "Notice",
  "message": "",
  "button_text": "OK",
  "date": ""
}
```

## Notice History / Bell Inbox

Use the top-level `notices` array when you want the bell page to show a scrollable history of Notices.

Add it beside `announcement`, `app_update`, and `documents`:

```json
"notices": [
  {
    "enabled": true,
    "id": "notice_2026_05_10",
    "title": "New CPG update",
    "message": "CPG has been updated. Open Guidelines to view the latest version.",
    "button_text": "OK",
    "date": "May 10, 2026"
  },
  {
    "enabled": true,
    "id": "notice_2026_05_08",
    "title": "CPR update",
    "message": "CPR Timer received UI and export improvements.",
    "button_text": "OK",
    "date": "May 08, 2026"
  }
]
```

Fields:

- `enabled`: set to `true` to show this Notice in the bell inbox.
- `id`: unique ID for this Notice. The app uses this to track read/unread state.
- `title`: title shown in the Notice card.
- `message`: main Notice text.
- `button_text`: kept for compatibility with the popup Notice. The inbox currently uses Mark read.
- `date`: publish date shown under the title. Use this on every Notice.

If `notices` is missing or empty, the app still falls back to the single `announcement` object when it is enabled.

Recommended setup:

- Keep the newest important Notice in `announcement` if you want it to auto-popup.
- Add the same Notice to `notices` so it remains visible in the bell history.
- Keep older Notices in `notices` as long as you want users to be able to review them.

Example using both:

```json
"announcement": {
  "enabled": true,
  "id": "notice_2026_05_10",
  "title": "New CPG update",
  "message": "CPG has been updated. Open Guidelines to view the latest version.",
  "button_text": "Got it",
  "date": "May 10, 2026"
},
"notices": [
  {
    "enabled": true,
    "id": "notice_2026_05_10",
    "title": "New CPG update",
    "message": "CPG has been updated. Open Guidelines to view the latest version.",
    "button_text": "Got it",
    "date": "May 10, 2026"
  }
]
```

The app removes duplicates by `id`, so the same Notice will not show twice.

### How Notice Appears In The App

The app startup dialog order is:

1. Terms & Conditions.
2. Notification permission alert.
3. Notice announcement.

The Notice will not appear before Terms are accepted.
The Notice will not appear before the notification alert is dismissed or completed.

If the user taps the Notice button, the app saves the current `announcement.id` locally as dismissed.
The same Notice will not auto-open again for that user.

The user can still view the same message manually by tapping the `Notice` button in the MainActivity status strip.

### When To Change The Notice ID

Change `id` every time you want the Notice to pop up again automatically.

Good IDs:

```json
"id": "cpg-update-may-2026"
```

```json
"id": "rsi-checklist-update"
```

```json
"id": "2026-05-05-guidelines"
```

```json
"id": "1"
```

The ID does not need to be incremented.
It only needs to be different from the previous announcement ID.

Important:

- New announcement that should pop up again = use a new `id`.
- Same announcement with small wording fix = keep the same `id`.
- Reusing an old dismissed `id` means users who dismissed it before will not see it automatically.

### New Lines In Notice Messages

Use `\n` for line breaks:

```json
"message": "CPG has been updated.\nPlease check the Guidelines section.\nThank you."
```

Do not press Enter inside the JSON string.
Using `\n` is safer and keeps the JSON valid.

### Notice Button In MainActivity

When `announcement.enabled` is `true` and both `id` and `message` are not empty:

- The `Notice` button appears in the MainActivity status strip.
- The `Notice` and `Updates` buttons share the full strip width.
- If Notice is disabled or missing, the `Notice` button hides.
- When Notice is hidden, the `Updates` button expands to full width.

### Bell Badge In MainActivity

The bell badge counts unread items from the Notice inbox.

- The badge uses `notices` from `ambulance_app_config.json`.
- If `notices` is empty, it can use the active single `announcement`.
- Tapping the bell opens the Notice inbox.
- Users can mark one Notice read or mark all Notices read.
- Read state is stored locally on the device.

The bell badge is not based on Firebase Console notification history.

### Testing Notice

To test the Notice:

1. Update the testing Gist or production Gist with:

```json
"announcement": {
  "enabled": true,
  "id": "test-notice-1",
  "title": "Notice",
  "message": "This is a test Notice.",
  "button_text": "Got it!"
}
```

2. Make sure `AppConfigRepository.kt` is pointing to the Gist you want to test.
3. Clear app data or change the Notice `id`.
4. Open the app.
5. Accept Terms if shown.
6. Dismiss or complete the notification alert.
7. The Notice should appear.

To test the bell inbox:

1. Add a `notices` array to the testing or production Gist.
2. Give each Notice a unique `id`.
3. Make sure each Notice has `enabled: true`.
4. Open the app after the config check interval passes, or clear app data for a fresh test.
5. Tap the bell icon beside the Ambulance title.
6. Confirm the Notices appear.
7. Tap Mark read or Mark all read and confirm the bell badge clears.

If the Notice does not appear:

- Check `enabled` is `true`.
- Check `id` is not empty.
- Check `message` is not empty.
- Check the app is fetching the Gist you edited, testing or production.
- Change the `id` to a new value if the old one was already dismissed.
- Remember the app checks the config based on `config_check_interval_hours`.

### Notice And Backup

Dismissed Notice IDs are stored locally in this SharedPreferences file:

```text
announcement_pref
```

Bell inbox read IDs are stored locally in:

```text
notice_inbox_pref
```

Terms acceptance is excluded from Android backup, but Notice dismissal is not currently excluded.
That means if Android restores app data, a dismissed Notice may stay dismissed after restore.
If you want all users to see the Notice again, change the Notice `id`.

## Firebase Console Push Notifications

Firebase Console notifications are still useful as reminders, but they are not the reliable source for the bell inbox.

Recommended Firebase Console message:

```text
New Ambulance notice available. Open the app and tap the bell.
```

Why:

- Firebase Console sends notification messages.
- When the device is locked, backgrounded, or the app is killed, Android/Firebase may display the notification directly.
- In that situation, the app may not receive the notification content.
- If the app does not receive it, it cannot add it to the in-app inbox.

So the reliable workflow is:

1. Add the Notice details to `notices` in `ambulance_app_config.json`.
2. Optionally set the newest important item as `announcement` if you want it to auto-popup.
3. Send a Firebase Console push only to tell users to open the app/check the bell.

Opening MainActivity clears visible non-CPR app notifications when possible, but the Notice inbox history comes from app config.

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

## CCP Pediatric Dosing Helper

CCP Pediatric medication dosing is driven by a separate helper JSON:

```json
"pediatric_dosing": {
  "enabled": true,
  "helpers": [
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
  ]
}
```

Current helper file:

```text
https://docs.niwashibase.com/helpers/ccp_pediatric_dosing_helper.json
```

Local working copy:

```text
TEMP/ccp_pediatric_dosing_helper.json
```

Bundled Android fallback:

```text
app/src/main/assets/ccp_pediatric_dosing_helper.json
```

To update CCP pediatric dosing:

1. Edit and validate `TEMP/ccp_pediatric_dosing_helper.json`.
2. Upload the helper JSON to the docs URL.
3. Copy the same validated helper into `app/src/main/assets/ccp_pediatric_dosing_helper.json` before building a release, so the app has a safe offline fallback.
4. Increase the helper top-level `"version"` when dose content changes.
5. Increase top-level `"schema_version"` only when the contract/structure changes in a way the app must explicitly support.
6. Update the matching `schema_version` and `version` inside `ambulance_app_config.json`.
7. Save/upload app config.

The Android app validates that:

- `helper_type` is `ccp_pediatric_dosing`.
- helper `schema_version` equals the app-config helper `schema_version`.
- helper `version` equals the app-config helper `version`.
- the helper contains at least one medication.

If validation or download fails, the app uses cached helper data if available, otherwise it uses the bundled asset fallback.

For full CCP helper editing rules, use:

```text
TEMP/README_CCP_PEDS.md
```

## Safe Rules

- Do not rename `schema_version`.
- Do not rename `config_check_interval_hours`.
- Do not rename `app_update`.
- Do not rename `announcement`.
- Do not rename `notices`.
- Do not rename `documents`.
- Do not rename `pediatric_dosing`.
- You can add new fields later. The app ignores fields it does not know.
- Increase a version when you want the app to refresh that item.
- Keep version numbers simple, for example `2.1`, `2.2`, `"4.1"`.
- For Notice messages, change `announcement.id` when you want the popup to show again.
- For bell inbox Notices, every item in `notices` needs a stable unique `id`.
- For CCP Pediatric dosing, helper `schema_version` and `version` must match the values in app config.
