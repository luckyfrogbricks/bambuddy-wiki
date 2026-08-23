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

!!! tip "Optional: AIM symbology IDs for stricter classification"
    If your scanner can be configured to transmit **AIM symbology identifiers** (a `]E0`-style prefix; on the DE2120 it's a configuration barcode in the manual, often labeled *Code ID* → *AIM ID*), SpoolBuddy uses it to tell a true retail EAN/UPC read apart from, say, a Code 128 label whose digits merely *look* like a barcode number — so only genuine retail barcodes land in a spool's GTIN field. Entirely optional: without the prefix, classification falls back to the (already checksum-validated) heuristic.

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

However the code arrives (hardware scan or **Enter Code Manually**), Bambuddy classifies it — a retail **GTIN** barcode, an Amazon **ASIN**, a manufacturer **SKU**/article number, or your own custom code — and resolves it in this order:

1. **Your own inventory** — any code stored on one of your rolls (its GTIN, SKU, ASIN, or a custom code) matches instantly and offline; the most recently added roll provides the template, so your latest edits carry forward. Archived spools count too.
2. **[Open Filament Database](https://openfilamentdatabase.org)** — a community database of retail filament barcodes. Queried only when your own inventory has no match, and only if the lookup setting (below) is enabled.
3. **[SpoolmanDB-Community](https://github.com/Icezaza2543/SpoolmanDB-Community)** — a much broader community filament catalog with a smaller set of attached barcodes. Checked when the Open Filament Database has no match, under the same setting.

The confirm screen labels where the match came from ("Matched in your inventory", "Matched in Open Filament Database", …) so you know how much to trust the prefilled fields. A **Location** row pre-selects wherever your last-added spool went (tap it to pick another location or create one in place), and one tap creates the spool with the RFID tag, weight, and codes on a single record. When the databases also know the same package's sibling code (the SKU printed next to the barcode, or vice versa), it's stored too — so either code on the box matches this roll on a future scan.

After **Add to Inventory**, lifting the roll off the scale within a few seconds turns the card's Close button into a short **"Closing in 3…"** countdown — tap it to close now, or the **×** to keep the card open.

### No match? Find This Filament

Barcode coverage is spotty per brand, and Amazon boxes often carry an Amazon code instead of the retail barcode (check the box for a second barcode). When no database recognizes a code, the primary action is **Find This Filament**: search your inventory and both community catalogs by text (e.g. `polymaker charcoal`). Each result is one **purchasable package** — the 0.5 kg, 1 kg, and 3 kg boxes of the same color appear as separate rows with their weights — and the disclosure arrow on a row expands its full details, including every known code with a per-code refill marker, so near-identical results are tellable apart. Select a row and tap **Confirm**: the scanned code is linked to it, so the *next* scan of that code resolves instantly. **Add Basic Spool** and manual entry remain as fallbacks, and the scanned code is kept on the spool either way.

### Refill packs

Refill boxes (the bare coil sold without a spool) are detected automatically where the community databases know the refill barcode, and the flow pre-selects the refill option — with a warning if the measured weight doesn't look right for the choice (a "refill" that weighs like a full spool, or vice versa). Spools bought as refills show a **Refill pack** badge in the inventory list and the spool editor; the badge records how the roll was purchased — mounting the coil on a reusable spool later is just a core-weight edit.

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

Spools gain editable **GTIN**, **SKU**, and **ASIN** fields (spool form → Additional section), all of them searchable — type a SKU like `17600` into the inventory search to find every roll of that product. An optional Inventory column shows a spool's codes, refill purchases carry the **Refill pack** badge, and the code fields round-trip through CSV import/export — see [Inventory → Barcodes](../features/inventory.md#barcodes). Desktop scanning hardware is not part of this feature.
