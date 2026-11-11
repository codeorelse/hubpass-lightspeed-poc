# Hub Pass · Lightspeed web extension POC

Proof of concept for Hub Pass member recognition in Lightspeed K-Series, styled like the Hub Pass kiosk on the Raspberry Pi. The page pairs the till with the Membership Service, polls for Hub Pass taps meant for this till, and applies the member's reward through the web extension JavaScript bridge (`pos_*` functions).

API: `https://api.dev.guestplatform.thesocialhub.app` (dev).

## What it does

1. **Checks the device token.** On start it reads `apiToken` with `pos_loadData`. No token means the pairing screen.
2. **Pairing.** A manager enters a pairing code (`XXXX-XXXX`) from the Membership Platform admin plus the door ID and area ID of the Hub Pass reader this till serves. The page calls `POST /pos/pair` with `{ code, deviceId, doorId, areaId, locationId?, deviceName? }` and stores the returned `hpd_…` token with `pos_storeData`. The token is only returned once, so the page reads it back to check it was saved.
3. **Polling.** While "Waiting for tap..." is shown, the page calls `POST /pos/pending-tap/claim` once a second with `Authorization: Bearer <token>`.
   - `204`: nothing waiting, keep polling.
   - `200`: shows the member and stops polling until **New tap**.
   - `409 already_claimed`: another till took it, keep polling.
   - `401`/`403`: token cleared, back to pairing.
   - `429`: pause; network errors back off up to 10 s.
4. **Recognised member.** The logo twirls into a green check, the member card shows name, segments, status and photo (fetched from `photoUrl` with the device token). Then, in order:
   - `pos_addExternalReference(tshId, "TSHID")`
   - `pos_setConsumer({ firstName, lastName })`
   - the first code in `discountCodes` as a global discount (the others are shown as "also eligible")
5. **Not recognised** shows a "Not recognised" card and sends nothing to the POS.
6. **Undo** removes the reward, **New tap** resumes polling, **Checkout** and **Done** hand back to the POS.

Every POS call and every non-204 API response goes to the call log on the page, with a copy button. Tokens and pairing codes are masked in the log.

## Extension URL

Add the device ID as a query parameter, filled in by Lightspeed:

```
https://codeorelse.github.io/hubpass-lightspeed-poc/?deviceId={deviceId}
```

Optional: `doorId`, `areaId` (prefill the pairing screen), `locationId`, `deviceName`, `userId` (fall back to `posContext`).

## Test settings

- **Discount method:** `pos_toggleGlobalDiscount` (default; reads the account first and only toggles when the code isn't on it yet) or `pos_applyGlobalManualDiscount` (code only, undocumented).
- **Clear stored token** / **Paste token:** switch between pairing and polling without a new pairing code.
- **Fake tap:** shows a local test member and applies the code in the settings field.

Opened outside Lightspeed, the page runs in demo mode with simulated `pos_*` calls; the API calls are real.

## Known blockers

- The dev API's CORS allowlist must include the origin that hosts this page (GitHub Pages is currently refused).

## Notes

- The Social Hub's typeface (Hour Sans) is proprietary and deliberately not included in this public repo. The page uses it only if it is installed on the device and otherwise falls back to a condensed system font.
- Fake-tap photo: placeholder portrait from Unsplash (Unsplash licence), embedded in the page.
- `pos_setConsumer` is called without `identifier`: that field references a Lightspeed consumer record UUID.
- POS barcode and NFC scans are logged but not used for recognition yet.
