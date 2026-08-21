---
title: Barcode Scanner
description: Add spools by scanning the box barcode with an optional USB scanner
---

# :material-barcode-scan: Barcode Scanner

SpoolBuddy can use an **optional USB barcode scanner** to identify new filament as you add it: place the roll on the device, scan the box's retail barcode, and the spool's brand, material, color, temperatures, and weight are filled in automatically — combined with the NFC tag and the measured weight in a single add-to-inventory flow, entirely on the touchscreen.

---

## :material-usb-port: Hardware

Any USB barcode scanner that acts as a keyboard (**USB HID keyboard emulation** — the default mode for nearly all of them) works. Connect it to a free USB port on the Pi.

- The scanner is auto-detected, and hot-plugging is supported — a missing scanner never blocks the daemon from starting.
- SpoolBuddy claims the scanner **exclusively**, so scans are decoded directly from the input device and never "typed" into the on-screen UI.
- A scan wakes the display if it has blanked.

!!! tip "Hands-free scanning"
    A scanner with a presentation / auto-sense mode (it fires when a barcode is presented, no trigger pull) makes the flow completely hands-free — set the roll down, hold the box up, done. See [Hardware Requirements](hardware.md#optional-usb-barcode-scanner) for a tested pick.

### Enabling and testing

Open **Settings → Scanner** on the SpoolBuddy touchscreen:

| Element | Description |
|---------|-------------|
| **Use barcode scanner when adding spools** | Per-device toggle. When on (and a scanner is detected), **Add to Inventory** opens the scan flow instead of the basic quick-add dialog. |
| **Detected hardware** | Shows whether a scanner is present and connected. |
| **Scan to test** | Live test row — scan any barcode and the decoded value appears here with a timestamp. |

---

## :material-line-scan: The scan flow

With the scanner enabled, tapping **Add to Inventory** on the Dashboard opens **Scan Barcode to Add**. The tag and weight your SpoolBuddy has already captured are shown as chips, and the screen waits for a scan.

Scanning is order-independent: you can also scan **before** the roll lands on the scale (or for a roll with no NFC tag) — the resolved filament shows immediately and the tag/weight chips fill in live when the roll arrives.

However the code arrives (hardware scan or **Enter Code Manually**), Bambuddy resolves it in this order:

1. **Your own inventory** — a barcode you've scanned before matches instantly and offline, including *sibling* codes learned from earlier scans (the refill-pack barcode, other package sizes, the manufacturer SKU). Archived spools count too.
2. **[Open Filament Database](https://openfilamentdatabase.org)** — a community database of retail filament barcodes. Queried only when your own inventory has no match, and only if the lookup setting (below) is enabled.
3. **[SpoolmanDB-Community](https://github.com/Icezaza2543/SpoolmanDB-Community)** — a much broader community filament catalog with a smaller set of attached barcodes. Checked when the Open Filament Database has no match, under the same setting.

The confirm screen labels where the match came from ("Matched in your inventory", "Matched in Open Filament Database", …) so you know how much to trust the prefilled fields, and one tap creates the spool with the RFID tag, weight, and barcode on a single record.

### No match? Find This Filament

Barcode coverage is spotty per brand, and Amazon boxes often carry an Amazon code instead of the retail barcode (check the box for a second barcode). When no database recognizes a code, the primary action is **Find This Filament**: search your inventory and both community catalogs by text (e.g. `polymaker charcoal`) and pick the right product — the scanned barcode is linked to it, so the *next* scan of that code resolves instantly. **Add Basic Spool** and manual entry remain as fallbacks, and the scanned code is kept on the spool either way.

### Refills

Refill boxes (the bare coil sold without a spool) are detected automatically where the community databases know the refill barcode, and the flow pre-selects the refill option — with a warning if the measured weight doesn't look right for the choice (a "refill" that weighs like a full spool, or vice versa). Refill spools show a **Refill** badge in the inventory list and the spool editor.

---

## :material-cog: Settings that affect scanning

| Setting | Where | Default | Effect |
|---------|-------|---------|--------|
| **Scan-to-Add Barcode Lookup** | Bambuddy web app → **Settings → Filament** | On | Allows lookups against the two community databases. With it **off**, nothing is ever downloaded — manual entry and matching against your own inventory still work fully. |
| **Use barcode scanner when adding spools** | SpoolBuddy → **Settings → Scanner** | On (when a scanner is present) | Per-device: switches Add to Inventory to the scan flow. |

!!! info "Offline and air-gapped installs"
    Community-database results are cached for 24 hours, and the Docker image ships with a built-in SpoolmanDB-Community snapshot — so scanning keeps working without connectivity: your own inventory always matches locally, and the snapshot provides catalog coverage even on an install that has never been online. External lookups only ever send the scanned code, nothing else.

Works with both the built-in inventory and [Spoolman](../features/spoolman.md) mode.

---

## :material-web: In the web app

Spools gain an editable **Barcode** field (spool form → Additional section), an optional Inventory column, the **Refill** badge, and barcodes round-trip through CSV import/export — see [Inventory → Barcodes](../features/inventory.md#barcodes). Desktop scanning hardware is not part of this feature.
