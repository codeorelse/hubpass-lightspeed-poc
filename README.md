# Hub Pass · Lightspeed web extension POC

Proof of concept for Hub Pass member recognition in Lightspeed K-Series, styled like the Hub Pass kiosk on the Raspberry Pi. No real reader, API or member data: the page shows "Waiting for tap...", fakes a tap after 2 seconds, and talks only to Lightspeed through the web extension JavaScript bridge (`pos_*` functions).

## What it does

1. Shows "Waiting for tap..." and fakes a tap after 2 seconds. Barcode and NFC scans on the POS also trigger it.
2. The logo twirls into a green check and a fake student appears: Emma de Vries, TSH ID `S7Q4M`, segment Student, Amsterdam City.
3. Calls, in order:
   - `pos_addExternalReference("S7Q4M", "TSHID")`
   - `pos_setConsumer({ firstName, lastName })`
   - the global discount `STU20` (Student 20%)
4. **Undo** removes the discount, **New tap** starts over, **Checkout** and **Done** hand back to the POS.

Every call and callback response goes to the call log on the page, with a copy button.

## Discount method

Switch under *Test settings*:

- `pos_toggleGlobalDiscount` (default): reads the current account first and only toggles when `STU20` isn't already on it.
- `pos_applyGlobalManualDiscount`: code only, no value. Undocumented; this POC is a way to test it.

## Lightspeed setup

1. Back Office: create a discount with code `STU20` (Student, 20%).
2. Back Office: add a web extension button pointing to the GitHub Pages URL of this repo.
3. On the POS: open a bill, press the button, wait 2 seconds.

Opened outside Lightspeed, the page runs in demo mode with simulated `pos_*` calls.

## Notes

- The Social Hub's typeface (Hour Sans) is proprietary and deliberately not included in this public repo. The page uses it only if it is installed on the device and otherwise falls back to a condensed system font.
- Student photo: placeholder portrait from Unsplash (Unsplash licence), embedded in the page.
- `pos_setConsumer` is called without `identifier`: that field references a Lightspeed consumer record UUID.
