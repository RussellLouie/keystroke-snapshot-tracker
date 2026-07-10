# Request Dashboard

A static web dashboard (hosted on GitHub Pages) for tracking requested **keystrokes** and **snapshots**.

- Requests are submitted through your **Microsoft Form**.
- Responses land in the Excel workbook in OneDrive/SharePoint.
- The dashboard reads those responses **live** via the Microsoft Graph API.
- Each 1go.team viewer **signs in with Microsoft**; **Status** and **Internal notes** are shared across the team (written back into a `Tracking` sheet in the same workbook).

No server required — everything runs in the browser.

---

## Files

| File | Purpose |
|------|---------|
| `index.html` | The entire app (UI + auth + Graph calls). Edit the `CONFIG` block near the top. |
| `README.md` | This guide. |

---

## Setup (three parts)

### 1. Register an Azure AD app (one time)

You (or IT) need an app registration in the **goteam1** Microsoft tenant.

1. Go to **portal.azure.com** → **Microsoft Entra ID** → **App registrations** → **+ New registration**.
2. Name: `Request Dashboard`.
3. Supported account types: **Accounts in this organizational directory only** (single tenant).
4. **Redirect URI** → platform **Single-page application (SPA)** → enter your GitHub Pages URL, e.g.
   `https://YOUR-ORG.github.io/request-dashboard/`
   (You can add `http://localhost:5500/` too for local testing.)
5. Click **Register**.
6. On the **Overview** page, copy the **Application (client) ID** and the **Directory (tenant) ID**.
7. Go to **API permissions** → **+ Add a permission** → **Microsoft Graph** → **Delegated permissions** → add:
   - `User.Read`
   - `Files.ReadWrite.All`
   Then click **Grant admin consent for goteam1** (needs an admin).
8. (If you added a redirect URI after registering: **Authentication** → confirm the SPA redirect URI(s) are listed and that **Access tokens** / **ID tokens** are fine — SPA uses the auth-code + PKCE flow, no secret needed.)

> `Files.ReadWrite.All` lets each signed-in user read/write files **they already have access to**. It does not grant access to files they otherwise couldn't open — so make sure everyone who should use the dashboard has access to the responses workbook (share the Form's Excel file with the team, e.g. a group).

### 2. Fill in the config

Open `index.html`, find the `CONFIG` block near the top of the `<script>`, and set:

```js
const CONFIG = {
  clientId:  "PASTE Application (client) ID",
  tenantId:  "PASTE Directory (tenant) ID",
  sharingUrl: "https://goteam1-my.sharepoint.com/.../Request Submission.xlsx",  // already filled with the link you shared
  responseSheet: "",        // leave blank to auto-detect the responses sheet
  trackingSheet: "Tracking",
  trackingTable: "TrackingTable"
};
```

The `Tracking` sheet and its table are **created automatically** the first time someone opens the dashboard, so you don't have to make them by hand.

### 3. Deploy to GitHub Pages

1. Create a repo (e.g. `request-dashboard`) and add `index.html` (and this README).
2. In the repo: **Settings → Pages → Build and deployment → Source: Deploy from a branch**, pick `main` / root.
3. Wait for the URL (e.g. `https://YOUR-ORG.github.io/request-dashboard/`).
4. Make sure that exact URL is registered as a **SPA redirect URI** in the Azure app (step 1.4).
5. Open the URL, click **Sign in with Microsoft**, and you're in.

---

## How it works

- **Read:** the sharing link is resolved to the workbook via Graph `GET /shares/{id}/driveItem`, then the responses sheet's used range is read.
- **Column mapping:** columns are matched to fields by header keywords (see `FIELD_MATCHERS` in `index.html`). If your form question titles differ, tweak the keyword lists there.
- **Write:** changing a Status dropdown or Internal note writes a row (keyed by the Form response **ID**) into the `Tracking` table via Graph. Everyone sees the same values on reload.
- **Status** values: `Pending`, `In progress`, `Done`.

---

## Troubleshooting

- **"Couldn't load data" / 403** — the signed-in user doesn't have access to the workbook, or admin consent wasn't granted. Share the Excel file with them and confirm consent.
- **Sign-in popup blocked** — allow popups for the GitHub Pages domain, or the redirect will still complete on retry.
- **Wrong columns showing** — adjust the keyword arrays in `FIELD_MATCHERS` to match your exact form question titles.
- **Nothing loads right after creating the form** — give SharePoint a few minutes; also make sure at least one response exists so the sheet has data.

---

*Built for internal 1go.team use.*
