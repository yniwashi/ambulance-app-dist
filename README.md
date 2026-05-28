# Ambulance App Config Guide

This guide explains the Android app config, app update workflow, Notices, and how document/helper versions trigger remote refreshes.

The static files referenced by app config live in:

```text
https://github.com/yniwashi/pdf-viewer
https://docs.niwashibase.com
https://api.niwashibase.com
```

## Live App Config

Current app config is served by the NiwashiBase API backed by Cloudflare R2:

```text
Production: https://api.niwashibase.com/api/v1/ambulance/app-config/production
Testing:    https://api.niwashibase.com/api/v1/ambulance/app-config/testing
Backup:     https://api.niwashibase.com/api/v1/ambulance/app-config/backup
```

Cloudflare R2 bucket:

```text
ambulance-app-configs
```

R2 object keys:

```text
production.json
testing.json
backup.json
```

Local TEMP drafts:

```text
/mnt/e/code assist/apps - work/android projects/TEMP/App Config/production.json
/mnt/e/code assist/apps - work/android projects/TEMP/App Config/testing.json
/mnt/e/code assist/apps - work/android projects/TEMP/App Config/backup.json
/mnt/e/code assist/apps - work/android projects/TEMP/App Config/ambulance_app_config.json
```

Legacy Gist app-config URLs were used before the API/R2 migration. Do not reintroduce Gist URLs unless intentionally rolling back.

## Release Checklist

Read this before every Android release.

1. Decide the release version:
   - increase `versionCode`;
   - increase `versionName`;
   - keep `versionName` aligned with the app-config `app_update.latest_version`.
2. Update the bundled fallback if needed:
   - `app/src/main/assets/ambulance_app_config.json`;
   - keep it reasonable for first launch with no cache/network.
3. Confirm live R2 app-config values:
   - production `access_gate.enabled`;
   - production `access_gate.disabled_access_type`;
   - `app_update.latest_version`;
   - `app_update.min_supported_version`;
   - `app_update.release_title`;
   - `app_update.release_message`;
   - `app_update.download_url`;
   - `app_update.force_update`;
   - Notices/announcement intended for release;
   - document/helper versions and URLs.
4. Confirm helper files:
   - Android lightweight helpers are uploaded to R2 `app-data/`;
   - iOS helper copies remain under `docs.niwashibase.com/helpers/` until iOS migrates;
   - PDFs and website icons still point to the intended docs host locations.
5. Upload app-config JSON files to R2:
   - `production.json`;
   - `backup.json`;
   - `testing.json` if testing should match.
6. Build the release APK.
7. Install the release APK on a test device.
8. Run the final smoke test:
   - app launch;
   - intended gate mode;
   - MainActivity dashboard;
   - one PDF/document path;
   - CPR open/start/stop;
   - Report Issue opens.
9. Upload the APK to the GitHub release asset used by `download_url`.
10. Verify the final `download_url` opens/downloads the new APK.
11. From an older installed version, confirm the update prompt points to the correct release.
12. If legacy versions still use the old Gist update JSON, update that Gist too and test the old-version update flow.

## How-To: Change The Live App Config

Use this for ordinary config-only changes such as update text, Notices, helper versions, access-gate mode, or config check interval.

1. Edit the local draft JSON first.
2. Validate that it is valid JSON.
3. Upload or overwrite the matching R2 object:

```text
production.json
testing.json
backup.json
```

4. Open the API endpoint in a browser/Postman and confirm it returns the new value:

```text
https://api.niwashibase.com/api/v1/ambulance/app-config/production
https://api.niwashibase.com/api/v1/ambulance/app-config/testing
https://api.niwashibase.com/api/v1/ambulance/app-config/backup
```

5. Open Android Admin Panel -> App Config and confirm:
   - resolved source is the expected source;
   - configured URL is the API endpoint;
   - updated values appear;
   - URL validation is accepted.

No APK rebuild is needed for a normal remote config change unless the bundled fallback or Android code also changed.

## How-To: Update The Bundled Fallback Config

Use this before building a release APK or when the APK must work correctly with no network and no saved cache.

1. Copy the intended fallback config into:

```text
app/src/main/assets/ambulance_app_config.json
```

2. Keep fallback notices/announcements conservative. For release fallback, avoid temporary testing messages.
3. Confirm fallback helper asset names match bundled files under:

```text
app/src/main/assets/
```

4. Build/install the APK and test once with no saved app data if fallback behavior matters.

Important: changing the bundled fallback requires a new APK. It does not affect already installed users until they install that APK.

## Lightweight Helper App-Data

Android lightweight helper/index URLs should use API/R2 app-data endpoints:

```text
https://api.niwashibase.com/api/v1/ambulance/app-data/{resource}
```

Examples:

```text
https://api.niwashibase.com/api/v1/ambulance/app-data/cpg-index
https://api.niwashibase.com/api/v1/ambulance/app-data/flowcharts
https://api.niwashibase.com/api/v1/ambulance/app-data/formulary
https://api.niwashibase.com/api/v1/ambulance/app-data/websites
https://api.niwashibase.com/api/v1/ambulance/app-data/as-call
https://api.niwashibase.com/api/v1/ambulance/app-data/hos-sites
https://api.niwashibase.com/api/v1/ambulance/app-data/analytics-config
https://api.niwashibase.com/api/v1/ambulance/app-data/rsi-checklist
https://api.niwashibase.com/api/v1/ambulance/app-data/ccp-pediatric-dosing
https://api.niwashibase.com/api/v1/ambulance/app-data/ap-pediatric-dosing
```

PDF documents and website icon files remain on `docs.niwashibase.com` for now. Do not move large PDFs to R2 without estimating bandwidth/request cost first.

Until the iOS webapp is migrated, helper updates must be published to both:

- `docs.niwashibase.com/helpers/` for iOS;
- R2 `app-data/` for Android.

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
- AS-Call helper URL/version
- HOS Sites helper URL/version
- Access Gate enabled state, base URL, fallback TTL, corporation-number length, and disabled-gate access type

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

### How-To: Change Config Check Frequency

1. Edit `config_check_interval_hours` in the R2 app-config JSON.
2. Upload the changed `production.json`, `testing.json`, or `backup.json`.
3. Existing installed apps will keep using their current cached config until their current check interval expires.
4. After the app successfully downloads the new config, the new interval is saved and used for future checks.

Example:

- If the old interval is `24` hours and you change it to `12`, a device may still wait up to the old 24-hour window before it downloads the new config.
- After it downloads the new config, the next check uses 12 hours.

To force a faster check for testing, use debug Admin Panel source switching/refresh, clear app data, reinstall, or wait for the current cached interval.

If the API/R2 app-config endpoint is unreachable, Android uses:

1. in-memory config if already loaded
2. saved cached config from the last successful fetch
3. backup app-config API endpoint if the active primary app-config URL fails
4. bundled fallback config inside the APK

The app does not replace saved config unless the remote JSON downloads and parses successfully.

## Access Gate Section

The root config can include:

```json
"access_gate": {
  "enabled": true,
  "base_url": "https://api.niwashibase.com/api/v1/ambulance/access-gate",
  "fallback_cache_ttl_sec": 86400,
  "corp_number_digits": 6,
  "disabled_access_type": "non_ambulance_staff"
}
```

Field meanings:

- `enabled`: when `true`, Android runs the Access Gate before MainActivity.
- `base_url`: Worker endpoint root for Access Gate requests.
- `fallback_cache_ttl_sec`: used only if a Worker response does not provide a TTL.
- `corp_number_digits`: corporation-number length accepted by the Android ambulance staff registration form.
- `disabled_access_type`: access mode used when `enabled` is `false`.

Allowed `disabled_access_type` values:

```text
non_ambulance_staff
ambulance_staff
```

Default behavior should stay `non_ambulance_staff` so disabling the gate does not accidentally grant protected document/internal-resource access. Use `ambulance_staff` only for intentional full-access maintenance/emergency mode.

### How-To: Temporarily Disable The Gate

1. Set:

```json
"enabled": false
```

2. Choose the disabled access mode:

```json
"disabled_access_type": "non_ambulance_staff"
```

or:

```json
"disabled_access_type": "ambulance_staff"
```

3. Upload the updated app config to R2.
4. Test app launch.

Recommended default is `non_ambulance_staff`, because it lets users open the app while still blocking protected document/internal-resource content. Use `ambulance_staff` only when you intentionally want full access while the gate is disabled.

## Remote URL Validation

Android validates remote URLs before using them.

App-config controlled document/helper URLs must use HTTPS and must be under `niwashibase.com` or one of its subdomains, for example:

```text
https://docs.niwashibase.com/helpers/as_call.json
https://docs.niwashibase.com/docs/cpg-81w9d1f.pdf
https://api.niwashibase.com/api/v1/ambulance/app-data/as-call
```

Rejected URLs do not replace working cached/bundled data. Admin Panel diagnostics show the configured URL/host and validation reason. User issue reports hide URLs/hosts but keep the validation state and reason so troubleshooting is still possible.

## Admin Panel And Issue Reports

The Android app has two diagnostic views:

- **Admin Panel:** full diagnostic view for trusted troubleshooting. It can show configured URLs, configured hosts, validation reasons, active source, cache state, versions, and recent errors.
- **Report Issue:** user-facing report flow. It asks for problem type, app area, and a short description, then shares a sanitized JSON report. User reports hide URLs, hosts, and domain names, but keep source/validation states so the issue can still be diagnosed later from Admin Panel if needed.

Resource source fields:

```text
using_source
active_source
config_source
resolved_source
```

Common values:

```text
live
cache
fallback
backup
memory
not_loaded
```

In debug builds, Admin Panel can switch app-config source for testing. Production builds use the production API app-config endpoint first, then the backup API app-config endpoint, cache, and bundled fallback as needed.

## Switching Between Production, Testing, And Backup

`AppConfigRepository.kt` contains three app-config URLs:

Production:

```kotlin
private const val PRODUCTION_APP_CONFIG_URL =
    "https://api.niwashibase.com/api/v1/ambulance/app-config/production"
```

Testing:

```kotlin
private const val TESTING_APP_CONFIG_URL =
    "https://api.niwashibase.com/api/v1/ambulance/app-config/testing"
```

Backup:

```kotlin
private const val BACKUP_APP_CONFIG_URL =
    "https://api.niwashibase.com/api/v1/ambulance/app-config/backup"
```

The backup app-config API endpoint is used only when the active primary app-config source cannot be downloaded or parsed.
Keep it as a copy of the production app config unless deliberately testing fallback behavior.
Admin Panel reports the active source and may show app-config URLs for troubleshooting. User Report Issue JSON does not expose app-config URLs.

Debug builds can switch between Build Default, Testing, Production, and Backup from Admin Panel and refresh app config immediately. Production builds use the build default production config path and only fall back to backup/cache/bundled when needed.

## Bundled Fallback

Android also ships a fallback config inside the APK:

```text
app/src/main/assets/ambulance_app_config.json
```

Fallback order:

1. Use memory config if already loaded and still inside the check interval.
2. Use saved cached config from the last successful remote fetch.
3. Try fetching the primary remote API app-config if the check interval has passed.
4. If the primary remote fetch fails or the JSON cannot be parsed, try the backup API app-config.
5. If both remote sources fail, keep using saved cached config.
6. If there is no saved cache, use bundled fallback from APK assets.

Important:

- Clearing app data removes saved cached config.
- Uninstalling removes saved cached config.
- Reinstalling brings back the bundled fallback because it is part of the APK.
- The API/R2 app-config should be the live source of truth.
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
3. Update R2 app-config `app_update.latest_version`.
4. Update `min_supported_version` if older versions should be blocked.
5. Update `release_title`, `release_message`, and `download_url`.
6. Set `force_update`.
7. Upload the updated app-config JSON to R2.
8. Test on a fresh install and an existing install.

Installed apps see the update after `config_check_interval_hours` passes, or sooner if they have no cached config.

## GitHub Release APK Update

APK distribution is hosted in:

```text
https://github.com/yniwashi/ambulance-app-dist
```

The app-config `app_update.download_url` currently points to the stable app update Worker URL:

```text
https://update.niwashibase.com/apk
```

That URL is served by the Cloudflare Worker named:

```text
ambulance-update
```

The Worker resolves the configured GitHub release tag and APK asset, then downloads/redirects the correct APK. Keep the app config URL stable as `https://update.niwashibase.com/apk`; update the Worker release fields each release.

Current known release pattern:

```text
Repository: yniwashi/ambulance-app-dist
Release: Ambulance App v2.1
Asset: Ambulance.v2.1.apk
Update Worker: ambulance-update
Public update URL: https://update.niwashibase.com/apk
```

Recommended GitHub release update steps:

1. Build the signed release APK.
2. Go to:

```text
https://github.com/yniwashi/ambulance-app-dist/releases
```

3. Open the current release, or draft a new release if the version is changing.
4. Use a clear release tag/name, for example:

```text
v2.1
Ambulance App v2.1
```

5. Upload the APK as a release asset.
6. Use a versioned asset filename so it is clear to humans, for example:

```text
Ambulance.v2.1.apk
```

7. Update the Cloudflare Worker `ambulance-update` release fields:

```javascript
tag: "v2.1",
assetName: "Ambulance.v2.1.apk",
forceFilename: "Ambulance.v2.1.apk",
```

8. Keep R2 app config pointing to the stable Worker URL:

```json
"app_update": {
  "download_url": "https://update.niwashibase.com/apk"
}
```

9. Upload the updated `production.json`, `backup.json`, and `testing.json` to R2 as needed.
10. Test the GitHub direct APK URL in a browser:

```text
https://github.com/yniwashi/ambulance-app-dist/releases/download/v2.1/Ambulance.v2.1.apk
```

11. Test the stable update URL in a browser:

```text
https://update.niwashibase.com/apk
```

It should download the new APK version.

12. Test the app update button from an older installed APK.

### How-To: Replace The APK Behind The Stable Update URL

Use this when `https://update.niwashibase.com/apk` is still downloading the previous version.

1. Confirm the GitHub release tag and APK asset name, for example:

```text
Tag: v2.2
Asset: Ambulance.v2.2.apk
```

2. Edit the Cloudflare Worker named:

```text
ambulance-update
```

3. Update only these release fields:

```javascript
tag: "v2.2",
assetName: "Ambulance.v2.2.apk",
forceFilename: "Ambulance.v2.2.apk",
```

4. Deploy the Worker.
5. Open:

```text
https://update.niwashibase.com/apk
```

6. Confirm the downloaded filename/version is the new APK.

The app config should normally keep:

```json
"download_url": "https://update.niwashibase.com/apk"
```

Do not change app config to a GitHub HTML release page.

GitHub release asset URL forms that usually work:

```text
https://github.com/yniwashi/ambulance-app-dist/releases/download/v2.1/Ambulance.v2.1.apk
```

Important:

- If using versioned APK asset names, update the `ambulance-update` Worker `tag`, `assetName`, and `forceFilename` every release.
- Make sure the GitHub release is published and not marked as pre-release unless intentionally testing.
- Do not point `download_url` to the GitHub HTML release page; it should point directly to the APK asset download.
- For current app config, `download_url` should stay `https://update.niwashibase.com/apk`, not the GitHub direct asset URL.

## Legacy Gist Update JSON

Older installed versions may still read the legacy Gist update JSON instead of the newer API/R2 app config. Until all active users are on the newer app-config flow, update the Gist every release.

For v2.1, the legacy Gist JSON was:

```json
[
  {
    "downloadLink": "https://update.niwashibase.com/apk",
    "version": 2.1,
    "title": "Ambulance App v2.1\n\nTo update follow these steps:\n\n1-Press Download to get the new version.\n\n2-Open My Files to install the new version (don’t use Google Chrome to install).",
    "body": "This version adds Ambulance Staff and Other User access, refreshed dashboard UI, improved CPR tools, updated pediatric tools, Shift Schedule, qSOFA, and safer reporting."
  }
]
```

After updating the Gist:

1. Open an older installed APK.
2. Confirm the update prompt shows the new version/title.
3. Tap Download.
4. Confirm it downloads from `https://update.niwashibase.com/apk`.
5. Install and confirm the app shows the new version.

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
  "index_url": "https://api.niwashibase.com/api/v1/ambulance/app-data/cpg-index"
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
https://api.niwashibase.com/api/v1/ambulance/app-data/cpg-index
https://api.niwashibase.com/api/v1/ambulance/app-data/sop-index
https://api.niwashibase.com/api/v1/ambulance/app-data/cpm-index
https://api.niwashibase.com/api/v1/ambulance/app-data/pat-index
```

To update a document:

1. Update the PDF on docs hosting if the PDF changed.
2. Update the index helper in R2 `app-data/`. Until iOS migrates, also update the docs helper copy.
3. Update `pdf_url` or `index_url` in app config if the filename changed.
4. Increase the document `version`.
5. Upload the updated app-config JSON to R2.
6. Test Android Guidelines/Search.

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
  "url": "https://api.niwashibase.com/api/v1/ambulance/app-data/rsi-checklist",
  "show_image": true
}
```

To update RSI:

1. Update R2 `app-data/rsi_checklist_js_android.html`. Until iOS migrates where relevant, keep docs helper copies in sync.
2. Increase RSI `version`.
3. Set `show_image`.
4. Upload app config to R2.
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
  "url": "https://api.niwashibase.com/api/v1/ambulance/app-data/ccp-pediatric-dosing",
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
  "url": "https://api.niwashibase.com/api/v1/ambulance/app-data/ap-pediatric-dosing",
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
  "url": "https://api.niwashibase.com/api/v1/ambulance/app-data/websites",
  "fallback_asset": "websites.json"
}
```

The Android helper route is:

```text
https://api.niwashibase.com/api/v1/ambulance/app-data/websites
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
- each website destination URL is a valid HTTPS browser link
- each remote icon URL is valid HTTPS under `niwashibase.com` or a subdomain

Remote update rule:

- Change `websites.version` for website additions/removals, title/category/subtitle changes, URL changes, enabled/disabled changes, order changes, and icon URL changes.
- Keep helper top-level `version` synchronized with app-config `websites.version`.
- Change `schema_version` only when the websites JSON contract changes and Android supports that change.
- Keep bundled fallback `assets/websites.json` updated before release builds.
- Website icons can be remote through `icon_url`; Android should cache icons and fall back gracefully if an icon fails.

## AS-Call Helper

AS-Call is configured under:

```json
"as_call": {
  "enabled": true,
  "schema_version": "0.1",
  "version": "0.1",
  "url": "https://api.niwashibase.com/api/v1/ambulance/app-data/as-call",
  "fallback_asset": "as_call.json"
}
```

The Android helper route is:

```text
https://api.niwashibase.com/api/v1/ambulance/app-data/as-call
```

Android uses the typed helper with `contacts` and lightly obfuscated `number_ref` values:

```json
{
  "helper_type": "as_call",
  "schema_version": "0.1",
  "version": "0.1",
  "contacts": [
    {
      "id": "scheduling",
      "enabled": true,
      "name": "Scheduling",
      "number_ref": "v1:Nzc5NTkwNzE"
    }
  ]
}
```

Android still accepts the older `addressbook` shape and plain `contacts[].number` during transition, but new helper edits should use `number_ref`.

Decoded numbers are accepted only if they contain normal phone characters: digits, `+`, `#`, `*`, spaces, hyphen, and parentheses.

Remote update rule:

- Change `as_call.version` for contact additions/removals, name changes, number changes, enabled/disabled changes, or order changes.
- Keep bundled fallback `assets/as_call.json` updated before release builds.
- Use `contacts[].number_ref` for new helper updates.

## HOS Sites Helper

HOS sites are configured under this app-config key:

```json
"hos_sites": {
  "enabled": true,
  "schema_version": "0.1",
  "version": "0.1",
  "url": "https://api.niwashibase.com/api/v1/ambulance/app-data/hos-sites",
  "fallback_asset": "hos_sites.json"
}
```

The Android helper route is:

```text
https://api.niwashibase.com/api/v1/ambulance/app-data/hos-sites
```

Remote update rule:

- Change `hos_sites.version` for site additions/removals, name/detail changes, enabled/disabled changes, status changes, order changes, or encoded location changes.
- Keep helper top-level `version` synchronized with app-config `hos_sites.version`.
- Keep bundled fallback `assets/hos_sites.json` updated before release builds.
- `location_ref` is light obfuscation, not encryption. It prevents casual readers from seeing plain coordinates but does not protect coordinates from app users or reverse engineers.

## Safe Rules

- Do not rename `schema_version`.
- Do not rename `config_check_interval_hours`.
- Do not rename `app_update`.
- Do not rename `announcement`.
- Do not rename `notices`.
- Do not rename `documents`.
- Do not rename `pediatric_dosing`.
- Do not rename `websites` after Android starts using it.
- Do not rename `as_call` after Android starts using it.
- You can add new fields later. The app should ignore unknown fields.
- Increase a version when you want Android to refresh that item.
- Keep version values simple, for example `2.1`, `2.2`, `"4.1"`.
- For Notice popup replay, change `announcement.id`.
- For bell inbox Notices, each Notice needs a stable unique `id`.
- For CCP/AP pediatric dosing, helper `schema_version` and `version` must match app config.
- For Websites, helper `schema_version` and `version` must match app config.
- For AS-Call, app-config `schema_version` and `version` control Android cache refresh. New AS-Call helpers should include matching top-level metadata and `contacts[].number_ref`.

## Quick Update Checklist

For app release:

1. Build APK.
2. Upload APK to GitHub release.
3. Update the `ambulance-update` Worker release fields if the APK tag/asset changed.
4. Update R2 app config `app_update`.
5. Update legacy Gist update JSON if old installed versions still need it.
6. Test `https://update.niwashibase.com/apk`.
7. Test update dialog from an older installed APK.

For document update:

1. Update PDF/index in `pdf-viewer`.
2. Upload Android index/helper copy to R2 `app-data/`.
3. Increase document `version` in app config.
4. Upload app config to R2.
5. Test Guidelines/Search.

For helper update:

1. Update helper in `pdf-viewer/helpers`.
2. Upload Android helper copy to R2 `app-data/`.
3. Increase matching helper/document version in app config.
4. Upload app config to R2.
5. Update Android bundled fallback if applicable before a release build.
6. Test the app feature.

For Notice:

1. Add to `notices`.
2. Add to `announcement` if popup is needed.
3. Use a new `id` if users should see it again.
4. Upload app config to R2.
5. Optionally send Firebase push as a reminder only.
