# Task Tracker

A responsive, single-file daily task and issue tracker with local storage, intelligent duplicate detection, dashboard metrics, filtering, and optional Google Sheets synchronization.

## Features

- Single-file application: `index.html`
- Mobile-friendly responsive interface
- Daily task and issue logging
- Local persistence using browser `localStorage`
- Monthly and keyword search filters
- Dashboard metrics for total, monthly, resolved, and duplicate entries
- Similar issue detection using text token similarity
- Optional Google Sheets synchronization through Google Apps Script
- Suitable for packaging in a WebView or HTML-to-APK builder

## Quick start

1. Open [`index.html`](./index.html) in a browser.
2. Enter an issue or task, its reason, and the resolution or action taken.
3. Click **Save entry**.
4. Entries are stored locally in the browser.

The application works without a backend. Google Sheets synchronization is optional.

## Configure Google Sheets synchronization

### 1. Create the spreadsheet endpoint

1. Create a Google Sheet.
2. Open **Extensions → Apps Script**.
3. Add the following Apps Script:

```javascript
function doPost(e) {
  const data = JSON.parse(e.postData.contents);
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheets()[0];

  if (sheet.getLastRow() === 0) {
    sheet.appendRow([
      'Date', 'Category', 'Issue', 'Reason', 'Resolution',
      'Status', 'Duplicate', 'Created At'
    ]);
  }

  sheet.appendRow([
    data.date || '',
    data.category || '',
    data.issue || '',
    data.reason || '',
    data.resolution || '',
    data.status || '',
    data.duplicate ? 'Yes' : 'No',
    data.createdAt || new Date().toISOString()
  ]);

  return ContentService
    .createTextOutput(JSON.stringify({ ok: true }))
    .setMimeType(ContentService.MimeType.JSON);
}

function doGet() {
  return ContentService
    .createTextOutput(JSON.stringify({ ok: true, message: 'Task Tracker API is running' }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

4. Deploy it as a web app:
   - **Execute as:** Me
   - **Who has access:** Anyone
5. Copy the deployment URL ending in `/exec`.
6. Open `index.html` and replace:

```javascript
const SHEET_ENDPOINT='https://script.google.com/macros/s/REPLACE_WITH_SCRIPT_ID/exec';
```

with your deployment URL.

## Test the endpoint

```bash
curl -X POST "YOUR_SCRIPT_URL" \
  -H "Content-Type: application/json" \
  -d '{"date":"2026-09-22","category":"Technical","issue":"Test issue","reason":"Test reason","resolution":"Test resolution","status":"Open","duplicate":false}'
```

To check that the endpoint is available:

```bash
curl "YOUR_SCRIPT_URL"
```

## Package as an Android APK

The app is self-contained and can be loaded into an Android WebView or HTML-to-APK builder.

1. Use `index.html` as the app entry point.
2. Enable Internet permission if Google Sheets synchronization is configured.
3. Enable WebView DOM storage/local storage.
4. Build and install the APK.

For offline use, local storage works without network access. Synchronization will resume only when the configured endpoint is reachable and a new entry is submitted.

## Customization

The main areas to customize are all in `index.html`:

- Update the color variables in the `:root` CSS block.
- Add or remove categories in the category `<select>` element.
- Adjust the duplicate similarity threshold in the `duplicate()` function.
- Change the Google Apps Script endpoint in `SHEET_ENDPOINT`.
- Add additional fields to the form and the sync payload.

## Data and privacy

Entries are stored in the browser's local storage for the current site/origin. Clearing browser or WebView data removes local entries. If Google Sheets synchronization is enabled, submitted entries are also sent to the configured Apps Script endpoint and stored in its associated spreadsheet.

## License

MIT
