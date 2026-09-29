---
name: gtin-identifier-auditor
owner: cartlift
category: Product feeds
description: Validates GTIN check digits, brand and MPN coverage and identifier_exists use across a feed, and flags rows it cannot verify instead of filling them in.
version: v2
license: MIT
updated: 2026-08-27
recommended: false
security_checked: true
url: https://ecomdly.com/skills/cartlift/gtin-identifier-auditor
raw: https://ecomdly.com/raw/cartlift/gtin-identifier-auditor.md
install: npx @ecomdly/cli add cartlift/gtin-identifier-auditor
---

# GTIN identifier auditor

Wrong identifiers quietly cost reach: the item shows, but Google cannot match it to the catalog. This skill checks every identifier by rule, not by eye.

## When to use
- Before launching a new supplier's range in Shopping.
- When Merchant Center reports "Limited performance due to missing identifiers" or "Invalid GTIN".

## Input
Feed rows with `id`, `item_group_id`, `brand`, `gtin`, `mpn`, `identifier_exists`, `condition`, and title. Optional: the supplier's own identifier list.

## Checks
1. **Format.** GTIN must be numeric, length 8, 12, 13 or 14 (GTIN-8, UPC-A, EAN-13, ITF-14). Strip spaces and hyphens; a leading zero dropped by a spreadsheet turns a UPC into 11 digits, flag it.
2. **Check digit.** From the right, excluding the check digit, weight digits 3,1,3,1…; check digit = (10 − sum mod 10) mod 10. Report failures with the computed digit.
3. **Reserved ranges.** Prefixes 020–029, 040–049 and 200–299 are internal/restricted-circulation numbers and are rejected; 978/979 are ISBNs (valid for books only).
4. **Uniqueness.** One GTIN on two different variants (other size or color) is an error. Same GTIN on duplicate rows is a feed duplication issue.
5. **Brand + MPN.** When no GTIN exists, `brand` and `mpn` together are required for branded goods. Brand must not be the store name unless the store makes the product.
6. **identifier_exists.** `no` only for custom, handmade, vintage or private-label items with no GTIN; `no` on an item that has a manufacturer GTIN gets it disapproved.

## Rules
- Never generate, look up by guess or "correct" a GTIN. A failed check digit is reported with the computed digit as a hint, not written into the feed.
- Rows where brand or product origin is unknown go to "needs owner input".
- Read-only; the output is a fix list for a human.

## Output format
```
| id | gtin | issue | suggestion |
| SKU-1182 | 5901234123458 | ok | |
| SKU-1183 | 590123412345 | 12 digits, check digit fails | verify with supplier |
| SKU-2210 | 0012345000065 | same GTIN as SKU-2209 (other size) | supplier sheet needed |
| SKU-3301 | — | identifier_exists=no but brand "Salomon" | find manufacturer GTIN |
Summary: 1 240 rows, 1 102 ok, 61 invalid, 77 missing; 14 need owner input.
```

## License
MIT
