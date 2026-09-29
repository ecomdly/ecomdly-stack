---
name: gtin-identifier-auditor
owner: cartlift
category: Product feeds
description: Audits every GTIN, MPN, brand and identifier_exists value in a product feed by GS1 rule (length, check digit, restricted prefixes, variant uniqueness) and lists the fix per row, without ever inventing a number. For feed and catalog managers.
version: v3
license: MIT
updated: 2026-09-29
recommended: false
security_checked: true
url: https://ecomdly.com/skills/cartlift/gtin-identifier-auditor
raw: https://ecomdly.com/raw/cartlift/gtin-identifier-auditor.md
install: npx @ecomdly/cli add cartlift/gtin-identifier-auditor
---

# GTIN identifier auditor

Produces a row-by-row identifier audit of a product feed (GTIN, MPN, brand, `identifier_exists`) with a verdict, the rule that failed, and the next action for each row. Wrong identifiers rarely cause loud errors: the item often still shows, but Google cannot match it to the catalog, so it loses reach, or a single `identifier_exists=no` on a branded product turns into a disapproval weeks later. The naive fix, "fill the empty GTINs and set everything else to no", is exactly what Google penalises. This skill validates every identifier by rule (length, GS1 check digit, reserved prefixes, uniqueness across variants), decides which of GTIN, brand + MPN, or `identifier_exists=no` applies to each product type, and never invents a number.

## When to use

- Before launching a new supplier's range in Shopping or on comparison sites.
- Merchant Center shows "Missing or incorrect GTIN", "Incorrect product identifier", "Limited performance due to missing identifiers" or similar.
- After a spreadsheet round-trip of the catalog (Excel drops leading zeros and turns long numbers into scientific notation).
- Before building a Heureka or Zboží.cz feed, where EAN drives product pairing.

## When not to use

- You need to find the correct GTIN for a product. This skill validates; it does not look up or generate GTINs. The source is the manufacturer, the supplier's data sheet, or the packaging barcode.
- The disapprovals are about price, availability, images or policy. Use `merchant-center-disapproval-fixer`.
- Only titles or attributes need work. Use `product-feed-optimizer` or `product-attribute-extractor`.

## Inputs

Required, one row per variant (the unit Google sees):

- `id`, `item_group_id`, `title`, `brand`, `gtin`, `mpn`, `identifier_exists`, `condition`.
- For apparel: `color`, `size` (needed to judge whether two rows are really different variants).
- `multipack` and `is_bundle` where used.

Where to get them: the primary feed file, or Merchant Center > Products > All products > download. Read GTINs as text: open CSV exports with the column typed as text, or parse the XML directly. If the file was already opened and saved in a spreadsheet, say so; leading-zero and exponent damage may already be baked in.

Optional:

- The supplier's identifier sheet (GTIN per supplier SKU) to cross-check.
- Private-label list: which products the store has made or rebranded for itself.
- Merchant Center issue export for the same IDs, to match audit findings with Google's issues.

If `brand` is missing for more than a few rows, stop and ask whether the catalog includes own-brand goods; the right fix differs completely.

## Best practices

### Format and validity

1. **Accepted lengths.** Google accepts GTINs of 8, 12, 13 or 14 digits: UPC (GTIN-12), EAN (GTIN-13), JAN (8 or 13), ISBN-13, ITF-14 (GTIN-14 for multipacks). Spaces and dashes are accepted but ignored; strip them before checking. Convert UPC-E (8-digit) to the 12-digit UPC-A and ISBN-10 to ISBN-13; if a book has both, submit only the 13-digit number.
2. **Check digit (GS1 mod 10).** Drop the last digit. From the right of the remaining digits, multiply by 3, 1, 3, 1…; sum; `check = (10 − (sum mod 10)) mod 10`. It must equal the last digit. The same algorithm works for all GTIN lengths. A failure means a typo or a truncated number; report the computed digit as a diagnostic hint, never as the corrected value, because a wrong digit elsewhere yields a different but equally valid-looking number.
3. **Reserved ranges.** Google rejects GTINs in restricted ranges (prefixes 02, 04 or 2) and coupon ranges (GS1 prefixes 05, 98 or 99). Apply the prefix test on the 13-digit form (left-pad a 12-digit UPC with one zero): in that form restricted means starting `02`, `04` or `2`, coupon means `05`, `98`, `99`. These are in-store and variable-measure numbers (a shop's own weighed-goods labels) or coupons, not trade identifiers. For 8- and 14-digit codes, flag for manual confirmation against GS1 rules rather than applying the 13-digit prefix test.
4. **Books.** `978` and `979` prefixes are ISBN-13 (Bookland); valid for books, suspicious on non-book categories.
5. **Leading-zero loss.** An 11-digit number that becomes check-digit-valid when a leading zero is added is almost certainly a UPC-A damaged by a spreadsheet. Report "probable lost leading zero, verify" with the padded value as a hint. Same for 12-digit values that validate as EAN-13 with a leading zero.
6. **Exponent damage.** Values like `5.90123E+12` are unrecoverable; the trailing digits are gone. Flag and re-export from source.
7. **Placeholder patterns.** All zeros, repeated digits, sequences, the same GTIN on hundreds of unrelated products, or a GTIN equal to the internal SKU are fake even when the check digit happens to pass.

### Uniqueness and variants

8. **One GTIN per variant.** Google: each product and variant (different colors or sizes) has its own GTIN. Two rows with the same GTIN and the same variant attributes for the same country and language are duplicates; Google names `condition` and `multipack` for all products and `color` and `size` for apparel as the variant attributes. Same GTIN on two different sizes means one of them is wrong.
9. **MPN behaves differently.** Different colors usually have different MPNs, but Google notes all sizes of an apparel product often share one MPN. Do not flag shared MPN across apparel sizes as an error.
10. **Multipacks and bundles.** Manufacturer-made multipack or bundle: use that pack's own GTIN, MPN and brand. Merchant-made multipack: use the individual product's identifiers and set `multipack`. Merchant-made bundle: use the main product's identifiers and set `is_bundle`.

### Which identifier applies

11. **Branded product with a manufacturer GTIN:** submit the GTIN (ideally with MPN and brand). This is the default; Google says products with missing or incorrect GTINs may have limited visibility and matching cannot be assured without one.
12. **Branded product without GTIN:** submit `brand` + `mpn`. Google makes MPN required for all products without a manufacturer-assigned GTIN, but only if you are sure it is correct; the manufacturer's part number, not a value you created (unless you are the manufacturer).
13. **Store brand or private label:** use the store name as `brand`. Google's brand spec says that when the merchant is also the manufacturer, submit the store name as brand and an MPN of your choice instead of `identifier_exists=false`. If the private-label product does carry a GTIN registered by the store, submit it.
14. **`identifier_exists=no`** only for new products that genuinely have no assigned identifiers: custom or one-of-a-kind goods, handmade items, and products made before GTINs existed (vintage, antiques, books before 1970). Google warns and can disapprove when `no` is set but evidence shows a GTIN exists (it compares with other merchants' offers). Setting `no` to silence a "missing GTIN" warning on a branded product is the most common self-inflicted error.
15. **Compatible and refurbished third-party goods:** brand and MPN of the company that actually built the product, never the OEM brand it fits.
16. **Used and vintage:** submit the manufacturer GTIN if the item has one, with `condition=used`.
17. **Brand hygiene.** One spelling per brand across the feed (`Salomon`, not `SALOMON`, `salomon`, `Salomon s.r.o.`). A brand equal to the store name on a product the store did not make is wrong.
18. **Ownership check.** Google points to GS1's GEPIR/Verified by GS1 to confirm which company a GTIN is licensed to. A valid check digit only proves the number is well-formed; if the company prefix belongs to an unrelated brand, the GTIN is wrong for this product.

### Other channels

19. Heureka accepts EAN in EAN-13 format and requires it only for books, textbooks, maps and guides, films, music and comics; Zboží.cz accepts 8, 12, 13 or 14 digits. A GTIN-12 that is fine for Google should be left-padded to 13 digits for Heureka. Hand the feed to `comparison-feed-mapper` after this audit.

## Process

1. **Normalise.** Trim, strip spaces and dashes, keep as string. Record the original value next to the normalised one.
2. **Classify each row** into exactly one verdict, in this order (first match wins):
   1. `damaged`: non-numeric, exponent notation, or length not in {8, 12, 13, 14} (with the lost-zero test from rule 5).
   2. `bad_check_digit`.
   3. `restricted_prefix` or `coupon_prefix`.
   4. `placeholder` (rule 7).
   5. `duplicate_variant` (same GTIN, different variant attributes) or `duplicate_row` (same GTIN, same attributes).
   6. `idex_conflict`: `identifier_exists=no` with a GTIN present or with a recognised third-party brand.
   7. `missing_identifier`: no GTIN and no brand + MPN, `identifier_exists` not set.
   8. `brand_issue`: brand empty, store name on third-party goods, inconsistent spelling.
   9. `ok`.
3. **Decide the fix per verdict:** damaged/bad digit/duplicate → re-source from supplier; restricted prefix → the item needs its trade GTIN, or remove `gtin` and use brand + MPN; idex conflict → remove `identifier_exists` or set true and supply the GTIN; missing → brand + MPN or, only if it qualifies under rule 14, `identifier_exists=no`; private label → store brand + own MPN.
4. **Group by supplier or brand.** Identifier errors cluster by source; one supplier sheet request fixes hundreds of rows.
5. **Summarise** counts per verdict and the share of rows with a valid identifier, and name the top three suppliers or brands by error count.

## Pitfalls and edge cases

- **GTIN-14 on single units.** An ITF-14 is a case or multipack code. On a single-unit listing it is usually the wrong code; ask for the unit EAN.
- **Regional codes.** Same product, different GTINs for different markets is normal; do not flag as a conflict if rows target different countries.
- **Repeated values.** Google allows up to 10 `gtin` values per product (for example UPC and ISBN on the same book). Validate each one separately.
- **Scanned barcodes on bundles.** Staff scanning the outer box of a merchant-made bundle often capture a component's EAN; check against rule 10.
- **Renumbering.** Changing a GTIN on a live product resets matching and can unpair it on comparison sites. Flag GTIN changes on top sellers for extra review.
- **`identifier_exists` language.** Values must be in English (`yes`/`no` or `true`/`false`), even in a Czech feed.

## Rules

- Never generate, look up by guess or "correct" a GTIN, MPN or brand. The computed check digit and the padded value are hints labelled as such, never written into the feed.
- Never recommend `identifier_exists=no` for a product with a recognised third-party brand unless the owner confirms it is custom-made or pre-GTIN.
- Rows where brand, origin or private-label status is unknown go to "needs owner input".
- Read-only. The output is a fix list; the agent does not edit the feed, the store or Merchant Center.
- Report counts that add up to the number of input rows.

## Output format

```text
Identifier audit: <feed name>, <n> rows, audited <YYYY-MM-DD>
Input note: <e.g. file was round-tripped through a spreadsheet; exponent damage possible>

| id | gtin (normalised) | brand | mpn | identifier_exists | verdict | detail | action | owner |

Summary
- ok: <n> (<%>)
- damaged: <n> | bad_check_digit: <n> | restricted/coupon: <n> | placeholder: <n>
- duplicate_variant: <n> | duplicate_row: <n> | idex_conflict: <n>
- missing_identifier: <n> | brand_issue: <n>
- needs owner input: <n>
Top sources of errors: <supplier/brand: count>, ...
Hints are diagnostic only; no value above should be written to the feed without supplier confirmation.
```

## Worked example

Example data: 8 rows from an outdoor store feed, CZ target.

```text
Identifier audit: shoptet-google.xml, 8 rows, audited 2026-09-29
Input note: supplier sheet for "TrailCo" was edited in a spreadsheet

| id | gtin (normalised) | brand | mpn | identifier_exists | verdict | detail | action | owner |
| SKU-1182 | 5901234123457 | Salomon | L41234 | | ok | | | |
| SKU-1183 | 5901234123458 | Salomon | L41235 | | bad_check_digit | computed check digit 7 (hint only) | ask supplier for unit EAN | purchasing |
| SKU-1184 | 12345000065 | TrailCo | TC-88 | | damaged | 11 digits; with leading zero 012345000065 validates (probable lost zero) | confirm with TrailCo, re-export as text | purchasing |
| SKU-2209 | 4006381333931 | Meindl | 2865-09 | | duplicate_variant | same GTIN as SKU-2210, sizes 42 vs 43 | get per-size EANs | purchasing |
| SKU-2210 | 4006381333931 | Meindl | 2865-09 | | duplicate_variant | see SKU-2209 | get per-size EANs | purchasing |
| SKU-3301 | | Salomon | | no | idex_conflict | branded product with identifier_exists=no | remove identifier_exists, add manufacturer GTIN | purchasing |
| SKU-4050 | 2001234567893 | Trail Store | TS-CAP-01 | | restricted_prefix | prefix 2 = in-store number; brand is the store | drop gtin, keep brand "Trail Store" + mpn TS-CAP-01 | marketing |
| SKU-5120 | 9780306406157 | | | | brand_issue | valid ISBN-13 on a book, brand empty | brand optional for books; add publisher as brand | content |

Summary
- ok: 1 (12.5 %)
- damaged: 1 | bad_check_digit: 1 | restricted/coupon: 1 | placeholder: 0
- duplicate_variant: 2 | duplicate_row: 0 | idex_conflict: 1
- missing_identifier: 0 | brand_issue: 1
- needs owner input: 0
Top sources of errors: Salomon: 2, Meindl: 2, TrailCo: 1
Hints are diagnostic only; no value above should be written to the feed without supplier confirmation.
```

Check-digit working for SKU-1183: body 590123412345; weights from the right 3,1,3,…: 5×3 + 4×1 + 3×3 + 2×1 + 1×3 + 4×1 + 3×3 + 2×1 + 1×3 + 0×1 + 9×3 + 5×1 = 15 + 4 + 9 + 2 + 3 + 4 + 9 + 2 + 3 + 0 + 27 + 5 = 83; (10 − 3) mod 10 = 7. The feed has 8, so it fails.

## Quality checklist

- Every row has exactly one verdict; verdict counts sum to the row count.
- GTINs were handled as strings; no value was altered except stripping spaces and dashes.
- Check digit computed for every 8/12/13/14-digit value.
- Prefix test applied on the 13-digit form; 8- and 14-digit codes flagged for manual prefix confirmation, not auto-rejected.
- No `identifier_exists=no` recommended for third-party branded goods.
- Private-label rows use store brand + own MPN, not `identifier_exists=no`.
- Apparel sizes sharing an MPN were not flagged.
- All hints labelled as hints; nothing presented as the corrected value.

## Sources

- https://support.google.com/merchants/answer/6324461 (GTIN attribute: lengths, restricted and coupon prefixes, variants, bundles, multipacks, private label)
- https://support.google.com/merchants/answer/6324482 (MPN attribute)
- https://support.google.com/merchants/answer/6324351 (brand attribute, store brand guidance)
- https://support.google.com/merchants/answer/6324478 (identifier_exists)
- https://support.google.com/merchants/answer/6098295 (How to fix: missing or incorrect GTIN)
- https://support.google.com/merchants/answer/9464748 (How to fix: incorrect product identifier)
- https://www.gs1.org/services/check-digit-calculator (GS1 check digit calculator)
- https://www.gs1.org/standards/id-keys/gtin (GS1 GTIN overview)
- https://sluzby.heureka.cz/napoveda/xml-feed/ (Heureka EAN rules)
- https://napoveda.sklik.cz/reklamy/xml-feed/specifikace/ (Zboží.cz feed specification, EAN element)

## License
MIT
