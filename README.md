# Hub Pass · Lightspeed web extension POC

Proof of concept for Hub Pass member recognition in Lightspeed K-Series. No real reader, API or member data: a tap loads a fake student, and the page talks only to Lightspeed through the web extension JavaScript bridge (`pos_*` functions).

## What it does

1. Waits for a tap: the **Simulate tap** button, or any scan from the POS barcode scanner or NFC listener.
2. Shows a fake student (Lucía Martínez, TSH ID `S7Q4M`, segment Student).
3. Calls, in order:
   - `pos_addExternalReference("S7Q4M", "HP")`
   - `pos_setConsumer({ firstName, lastName })`
   - the global discount `STU20` (Student 20%)
4. **Undo** removes the discount, **Go to checkout** and **Done** hand back to the POS.

Every call and callback response is written to the call log on the page, with a copy button, so results can be shared from the terminal.

## Discount method

Switch under *Test settings*:

- `pos_toggleGlobalDiscount` (default): reads the current account first and only toggles when `STU20` isn't already on it.
- `pos_applyGlobalManualDiscount`: code only, no value. Undocumented; this POC is a way to test it.

## Lightspeed setup

1. Back Office: create a discount with code `STU20` (Student, 20%).
2. Back Office: add a web extension button pointing to the GitHub Pages URL of this repo.
3. On the POS: open a bill, press the button, tap or press *Simulate tap*.

Opened outside Lightspeed, the page runs in demo mode with simulated `pos_*` calls.

## Notes

- The student photo is a placeholder portrait from randomuser.me; it falls back to initials if it can't load.
- `pos_setConsumer` is called without `identifier`: that field references a Lightspeed consumer record UUID.
