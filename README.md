# Gavirad Chemist Coverage Tracker

Live field tracker for Gavirad chemist coverage, built to run on GitHub Pages.
Same structure as the sweater campaign site: static pages here, data sync through
a Google Apps Script web app.

## Files

| File | Who uses it | What it does |
|---|---|---|
| `index.html` | MIO | Pick zone → region → territory, work the chemist call list, mark orders. Pushes each order to the cloud. |
| `admin.html` | Head office | Live dashboard. Filters Division → Zone → Region → Territory, summary cards, per-MIO and per-territory tables, Excel and CSV export. |
| `data/zones.json` | — | Index of 35 zones with their regions and territories. |
| `data/<Zone>.json` | — | One file per zone: map geography, regions, territories and the full chemist list. Loaded only when that zone is opened. |

## Deploy

1. Create a repository and upload everything in this folder, keeping the `data/` folder.
2. Settings → Pages → Source: `main` branch, `/ (root)` → Save.
3. Your links, after a minute or two:
   - MIO portal: `https://<user>.github.io/<repo>/`
   - Admin panel: `https://<user>.github.io/<repo>/admin.html`

## Apps Script

Paste this into the Sheet's Apps Script editor and deploy as a web app
(Execute as: Me · Who has access: Anyone), then redeploy after any edit.

```javascript
function doGet(e) {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sheet = ss.getSheetByName('Log');
  if (!sheet) { sheet = ss.insertSheet('Log');
    sheet.appendRow(['Timestamp','Rep','Zone','Region','Territory',
                     'Chemist Code','Chemist Name','Status','Marked At']); }
  var p = e.parameter;
  if (p.action === 'read') {
    var last = sheet.getLastRow();
    var rows = last > 1 ? sheet.getRange(2, 1, last - 1, 9).getDisplayValues() : [];
    return ContentService
      .createTextOutput(p.callback + '(' + JSON.stringify({ rows: rows }) + ')')
      .setMimeType(ContentService.MimeType.JAVASCRIPT);
  }
  sheet.appendRow([new Date(), p.rep || '', p.zone || '', p.region || '',
                   p.territory || '', "'" + (p.code || ''), p.name || '',
                   p.status || '', p.time || '']);
  return ContentService.createTextOutput('ok');
}
```

If you change the web app URL, update `ENDPOINT` in `index.html` and `admin.html`.

## Notes

- Orders are saved on the phone first and pushed when there is signal, so the
  tracker works offline in the field.
- The admin panel counts rows with status `ordered`. These are field claims,
  not invoiced sales — reconcile against the sales file before using them in a review.
- The repository is public on GitHub Pages. It contains chemist names, codes and
  Radiant sales values.
