# TradePulse

Shared clearance desk for customs clearing agencies in Ghana (Tema and Takoradi). Claude reads invoices into checkable lines, an admin signs off anything uncertain, and a consignment log keeps container numbers, arrival and free-time dates in one place.

TradePulse does **not** connect to ICUMS and never holds ICUMS logins. Agents file declarations themselves, in their own ICUMS session.

## What is in this repository

| Path | What it is |
| --- | --- |
| `tradepulse.html` | The whole app, one self-contained page built for claude.ai Artifacts |
| `docs/` | Planning documents from earlier drafts (blueprint v2, revision 3 upgrade notes, enhanced feature set, the original blueprint) |

## What the app does

- **Read invoice.** Upload photos or PDFs of a commercial invoice and packing list. Claude returns lines (description, HS code, quantity, unit, value) with a confidence score per field. The source pages sit beside the lines so the agent can check each one.
- **Sign-off.** Lines scoring 95 or more are green, 80 to 94 amber, below 80 red. A red field must be corrected by hand before the invoice can be submitted. If every line is green and the line values match the invoice total, the invoice clears on the agent's own review. Otherwise it goes to an admin, who approves it or returns it with a note.
- **Consignments.** Client, container or chassis number, BoE, terminal, arrival date and free-time end date. Container numbers are checked against their ISO 6346 check digit to catch typing mistakes. The free-time countdown uses the date the agent enters and predicts nothing.
- **Audit trail.** Every extraction, submission, approval, return and status change is logged with who and when. Export to CSV; the export shows a SHA-256 fingerprint of the saved file.

## Rules built into the logic

- An HS code that Claude suggested because none was printed is capped at amber, so it always reaches an admin.
- An HS code that is not 6 to 10 digits is capped at red.
- Editing a field marks it as edited by hand but does not remove the admin sign-off requirement.
- An approval applies only to the submission it was given for. If the agent edits and resubmits, the earlier approval no longer counts.
- Only admins (Editor or Owner access on the artifact) can write approvals. This is enforced by the database rule on the `approvals` collection.

## Running it

The app is a single HTML file that uses the claude.ai Artifact runtime. Publish it as an artifact with these capabilities:

```json
{
  "db": { "rules": [{ "path": "approvals", "read": "view", "write": "admin" }] },
  "user": { "scopes": ["profile"] },
  "sample": { "images": true },
  "downloads": true,
  "assets": { "upload": "interact" }
}
```

Share it from the artifact's Share menu: Editor access for admins, Contributor access for staff.

Outside the Artifact runtime the page opens in a local session: records stay in the tab, reading with Claude is unavailable, and you can still enter lines by hand.

## Data model

Shared collections in the artifact database: `jobs` (consignments with their extraction and source page ids), `approvals` (one document per job, written by admins only) and `audit` (append-only events).

## Known limits

- Confidence is Claude's own estimate of how clearly a page was read. It does not confirm that an HS code is correct for the goods. The agent stays responsible for classification.
- The audit trail is written from each person's session and an admin can remove records. Export it regularly.
- Source pages are visible to everyone who has access to the artifact.
- PDF reading loads a library on demand and has not been tested in every viewer. Photos are the safer input.
- The first read asks each user to allow Claude to use their own Claude account.

## Open questions before real client use

1. **Ghana Data Protection Act, 2012 (Act 843).** Using client data for any aggregated product needs explicit opt-in consent, not a clause in the terms of service. No such product is built here.
2. **ICUMS terms of use.** Whether scheduled automated login is permitted, even with the account holder's consent, is unconfirmed. Nothing in this app logs in to ICUMS.
3. **Real invoices.** Extraction accuracy has not been measured on real Ghanaian invoices. Test 15 to 20 before relying on the confidence scores.
