---
name: idoklad-api-operator
owner: marginmath
category: Agent tools
description: Lets an agent read and write iDoklad through API v3 for an online store: unpaid and overdue invoices, payments backed by bank records, Default-template drafts the owner approves, credit notes and accountant exports.
version: v1
license: MIT
updated: 2026-10-05
recommended: false
security_checked: true
url: https://ecomdly.com/skills/marginmath/idoklad-api-operator
raw: https://ecomdly.com/raw/marginmath/idoklad-api-operator.md
install: npx @ecomdly/cli add marginmath/idoklad-api-operator
---

# iDoklad API operator

Produces safe, reviewable work in one iDoklad agenda (company) through API v3: read-only reports (unpaid and overdue invoices, payments, contacts, store-to-iDoklad reconciliation, accountant exports), and for writes a preview the owner approves followed by an execution log of every document ID created or changed. The naive approach (hand-build a JSON invoice, POST it, mark it paid when the e-shop says "paid") fails in ways that end up in the VAT return: a hand-picked serial number collides with the numeric sequence, a guessed `VatRateType` puts 12% goods at 21%, an issued invoice gets edited instead of corrected by a credit note, and the payment POST sends the customer a thank-you e-mail when `SendPaymentConfirmation` is left at true, as it is in iDoklad's own sample request. **The key insight: let iDoklad fill in everything it owns. Start every new document from the matching `Default` endpoint, let `Recount` do the arithmetic, change only the fields the order dictates, and treat an issued document as final: anything after that is a new document (a payment or a credit note), never an edit.**

## When to use
- The owner asks what is unpaid or overdue, how much is outstanding per customer, or whether a specific order has been invoiced and paid.
- A bank statement or payment-gateway payout needs to be matched to issued invoices, and the matched payments recorded.
- An e-shop order (B2B, or B2C where the store issues invoices in iDoklad) needs a draft issued invoice, or an approved return needs a credit note (opravný daňový doklad).
- The accountant wants a period's issued and received invoices, credit notes and payments as a file or PDFs.

## When not to use
- Changing prices, stock or orders in the store itself: `shopify-admin-api-operator`.
- Reconciling GA4 or Google Ads revenue with backend revenue: `revenue-discrepancy-reconciler`; this skill only matches store orders and payouts to iDoklad documents.
- Deciding whether a return or claim is accepted: `warranty-claim-handler`, `returns-reason-analyzer`; this skill issues the credit note after the decision.
- Writing to customers about an order or a payment: `order-status-reply-drafter`.
- Marketplace fee and margin analysis: `marketplace-profitability-check`; weekly reporting: `weekly-store-kpi-report`.
- Any VAT, OSS or bookkeeping judgement: the owner's accountant.

## Inputs
| input | definition | typical source |
|---|---|---|
| Credentials | `client_id` and `client_secret` (and for the client credentials flow also `application_id`), read from environment variables at run time | iDoklad, Nastavení → Aplikace → API → Vygenerovat; `application_id` from the Developer portal |
| Agenda facts | VAT status (`VatRegistrationType`), default currency, rounding setting | `GET /v3/Account/CurrentAgenda` |
| Plan and quota | iDoklad plan and the monthly API request count | owner; iDoklad, Nastavení → Aplikace → API |
| Order or payment evidence | order lines with gross or net prices and the store's VAT rate per product, delivery date; for payments a bank or gateway record with amount, date and reference | store export, bank statement, gateway payout report |
| Accountant's conventions | which numeric sequence, which date is DUZP for shipped orders, how OSS and foreign currency are handled | the accountant, in writing |
| The task | what to read or create, for which period or documents | the owner |

Optional: a saved invoice template ID (`templateId` for `IssuedInvoices/Default`), tag IDs, the owner's log location.

Missing input → stop and ask. Never guess a VAT rate, a DUZP, a numeric sequence or a payment, and never fill a gap with "typical" values.

## Best practices

### Access
1. **v3 only, one base URL.** Requests go to `https://api.idoklad.cz/v3/...` with `Authorization: Bearer <token>`; JSON only. API v2 is switched off on 5 October 2026, so any v2 snippet found online is dead code.
2. **Pick the right OAuth2 flow.** For the owner's own agenda use client credentials: `POST https://identity.idoklad.cz/server/v2/connect/token` with `grant_type=client_credentials`, `application_id`, `client_id`, `client_secret`, `scope=idoklad_api`. Third-party apps acting for many users use authorization code (`/server/connect/authorize`, then `/server/connect/token` with `grant_type=authorization_code`, a one-time `code`, scope `idoklad_api offline_access`); each token response then carries a new refresh token that must replace the old one. Read `expires_in` from the response and refresh before it runs out; do not hard-code a lifetime.
3. **Secrets stay in the environment.** Read credentials from variables such as `IDOKLAD_CLIENT_ID`, `IDOKLAD_CLIENT_SECRET`, `IDOKLAD_APPLICATION_ID`; check presence, never print values, never write tokens to files, logs, previews or chat. A secret that appears in output is compromised; the owner regenerates it.
4. **API access depends on the plan.** iDoklad's price list includes API access in Oblíbený (7,500 requests a month) and Prémiový (75,000); the quota is per agenda and shared by all its users. HTTP 402 means an expired or insufficient plan; stop and tell the owner.

### Reading efficiently
5. **Page, filter, select.** `page` and `pagesize` (default 20, maximum 200); lists return `TotalItems` and `TotalPages`. Filters: `filter=(Field~op~value~and~Field~op~value)` with `eq`, `!eq`, `lt`, `lte`, `gt`, `gte`, `ct`, `!ct`; text values may be sent as `Field~eq:base64~<encoded>`; enums accept names (`PaymentStatus~eq~Unpaid`). Sort: `sort=DateOfIssue~desc`. Add `select=` to return only the fields you need.
6. **Filter only on allowed columns.** Issued invoices filter on `Id, CurrencyId, DateLastChange, DateOfIssue, DateOfPayment, DateOfMaturity, Exported, Description, NumericSequenceId, DocumentNumber, ConstantSymbolId, PartnerId, IsPaid, PaymentStatus, RecurringInvoiceId, NickName, DateOfTaxing, TagIds`. `VariableSymbol` and `OrderNumber` are not filterable there (received invoices do allow `VariableSymbol`), so match store orders by reading the period and joining locally.
7. **Respect the limits.** 200 requests a minute (HTTP 429 above that) plus the monthly plan quota. Use `select`, pagesize 200 and cached code lists (`VatRates`, `Currencies`, `PaymentOptions`, `NumericSequences`); on 429, wait for the next minute and retry reads only.

### Creating documents
8. **Default first, always.** `GET /v3/IssuedInvoices/Default` (also `ProformaInvoices/Default`, `Contacts/Default`, `CreditNotes/Default/{invoiceId}`, `IssuedDocumentPayments/Default/{documentId}`) returns a prefilled model from the agenda's settings: `NumericSequenceId`, `DocumentSerialNumber`, `CurrencyId`, `PaymentOptionId`, dates, bank details. Change only what the order dictates and POST the rest unchanged.
9. **Never number by hand.** The sequence's `LastNumber + 1` is the next serial number. Keep `DocumentSerialNumber` from Default; if the owner needs a specific sequence, check it with `GET /v3/NumericSequences/DocumentNumbers/{documentType}`, which returns `IsUnique`. Gaps and duplicates are an accountant's problem you created.
10. **VAT type from the code list, not from memory.** Map each rate with `GET /v3/VatRates` filtered by `CountryId` and validity on the `DateOfTaxing`; send `VatRateType` (`Reduced1`=0, `Basic`=1, `Zero`=2, `Reduced2`=3) plus `VatRate`. Czech law currently has 21% and 12% (zákon o DPH § 47); which goods take which rate comes from the store's product data and the accountant, never from the agent. Error 150/151 means a rate not valid on the DUZP.
11. **Three dates, three meanings.** `DateOfIssue` is the issue date, `DateOfTaxing` is DUZP (the date of the taxable supply; for goods the day of delivery, § 21), `DateOfMaturity` is the due date. A tax document must be issued within 15 days of the day the tax obligation arose (§ 28(8)), and must state DUZP where it differs from the issue date (§ 29(1)(h)). Which event counts as delivery for the store is the accountant's call; record it in the inputs.
12. **Let iDoklad compute totals.** `POST /v3/IssuedInvoices/Recount` (also for credit notes) returns per-item `Prices` and `Prices.VatRateSummary`. iDoklad does not document its rounding algorithm; a rounding line (`ItemTypeRound`) depends on the agenda's `RoundingDifference` and the payment option's `IsRounded`. Show Recount's numbers in the preview, not your own.
13. **Corrections are credit notes.** Start from `GET /v3/CreditNotes/Default/{invoiceId}`; `CreditedInvoiceId` and `CreditNoteReason` are required, matching the original number and reason that § 45 requires on a corrective document, which must reach the customer within 15 days (§ 42(5)). Do not PATCH an issued invoice's amounts; if it was already exported, any edit flips `Exported` to `Changed` in the accountant's sync.
14. **Contacts carry behaviour.** `SendReminders` on a contact makes iDoklad send automatic reminders. Search before creating (`IdentificationNumber`, `VatIdentificationNumber`, `Email`, `CompanyName` are filterable) and set `SendReminders` only as the owner says.

### Payments
15. **Evidence, then a check, then a record.** A payment needs a bank or gateway record (date, amount, reference). First read existing payments for the document (`GET /v3/IssuedDocumentPayments?filter=(InvoiceId~eq~{id})`): a payment may already exist from iDoklad's bank-statement pairing or a colleague, and a second record makes the invoice `Overpaid`. Then `GET .../Default/{documentId}`, then `POST /v3/IssuedDocumentPayments` with `InvoiceId`, `PaymentAmount`, `PaymentOptionId`, `DateOfPayment`, and `SendPaymentConfirmation: false`.
16. **Partial is normal.** Record the amount actually received; `PaymentStatus` becomes `PartialPaid`, outstanding = `Prices.TotalWithVat − Prices.TotalPaid`. `PUT /v3/IssuedDocumentPayments/FullyPay/{id}` books the full amount regardless of what arrived, so use it only when the evidence equals the total.
17. **Tax documents on payments are an accountant decision.** `CreateIssuedTaxDocument` (daňový doklad k přijaté platbě, for advance payments on proformas) stays as the accountant instructs.

## Process
1. **Confirm setup**: credentials present (names only), token obtained, `GET /v3/Account/CurrentAgenda` read (VAT status, currency, rounding), plan quota noted.
2. **Read**: pull the documents for the task with filters and `select`; for overdue use `DateOfMaturity~lt~<today>~and~PaymentStatus~!eq~Paid` and drop `Overpaid` rows locally.
3. **Reconcile** (if asked): join store orders and payout lines to invoices by `OrderNumber`, `VariableSymbol` or `Description` locally; list unmatched on both sides with amounts.
4. **Prepare** each write: Default → change order fields → Recount → preview (Output format) with DUZP, sequence, VAT summary and any flag.
5. **Approve**: show the preview and wait for an explicit yes; a changed preview needs a new yes.
6. **Execute** one document, then check `IsSuccess`, `StatusCode`, `ErrorCode`, `Message`, and re-read by `Id`; continue only if it matches the preview.
7. **Log** every created or changed `Id`, `DocumentNumber`, time and before/after values.
8. **Export** (if asked): list endpoints with `select` to CSV, PDFs via `GET /v3/Reports/IssuedInvoice/{id}/Pdf` (also `CreditNote`, `ReceivedInvoice`, `ProformaInvoice`); `Data` is a string; iDoklad encodes files as Base64 (a zip when `compressed=true`), so decode it and confirm the PDF opens.
9. **Run the quality checklist.**

## Pitfalls and edge cases
- **HTTP 200 is not the whole answer.** Every response is an ApiResult with `Data`, `IsSuccess`, `Message`, `StatusCode`, `ErrorCode`; read all of them. 400 is a validation error, 404 not found, 401 a dead token, 403 a needed manual downgrade.
- **Sample values are not defaults.** iDoklad's sample bodies show `SendPaymentConfirmation`, `SendToPartner` and `HasVatRegimeOss` set to true; copying a sample sends mail or switches the VAT regime.
- **OSS and foreign currency.** `HasVatRegimeOss`, a non-domestic `CurrencyId`, `ExchangeRate` and `ExchangeRateAmount` change how VAT is reported; § 29(1)(l) requires the VAT amount in Czech currency (the `...Hc` fields). Prepare such documents only with the accountant's written rule; error 119 means a malformed OSS request.
- **Exported documents.** `Exported` = 1 means the accountant's software has it. Never change it or toggle it via `PUT /v3/Batch/Exported` unless the accountant asks.
- **Store ≠ iDoklad totals.** Shipping, discounts and gateway fees often sit on different lines; a payout is net of fees, so match per order, not per payout total.
- **Non-VAT payers.** If `VatRegistrationType` is `NotVatPayer`, documents carry no VAT; do not add rates.
- **E-mail limits.** Sent mails count toward a daily cap (HTTP 429, error 122); another reason the agent never sends.
- **Timeouts on POST.** Re-read the list for the partner and date before retrying, or you create a duplicate number.

## Rules
- Reads need no approval; every POST, PATCH or PUT needs an owner-approved preview built from the Default template and Recount.
- Never call DELETE on any document, payment or contact, never `FullyUnpay`, and never edit an issued or exported document's amounts or dates.
- Never call any `/v3/Mails/...` endpoint (invoices, reminders, payment confirmations) and always send `SendPaymentConfirmation: false`. Sending to customers is the owner's act in iDoklad.
- Never record a payment without a bank or gateway record matching amount and reference, and the owner's approval.
- Never choose VAT rates, DUZP rules, OSS treatment or exchange rates; they come from the store's data and the accountant.
- Never print, log or store credentials or tokens, and never send iDoklad data anywhere except to the owner.
- Stop on the first unexpected error and report it with `ErrorCode` and `Message`.

## Output format
```
IDOKLAD PREVIEW · agenda <name> · prepared <date time, tz> · status PENDING APPROVAL
Document: <issued invoice | credit note to <DocumentNumber> | payment> · from Default (template <id or none>)
Partner: <name> (PartnerId <id>, IČO <n>, DIČ <n>) · order <store order no.>
Sequence: NumericSequenceId <id> · DocumentSerialNumber <n> (from Default) · currency <code>
Dates: DateOfIssue <d> · DateOfTaxing/DUZP <d> (<rule from accountant>) · DateOfMaturity <d>
| # | item | qty | unit price | price type | VAT type/rate | net | VAT | gross |
VAT summary (from Recount): <rate>: net <x> VAT <x> gross <x> · ... · total <x> · rounding line <x or none>
Payment (if any): amount <x> · date <d> · evidence <bank/gateway ref> · SendPaymentConfirmation false
Flags for accountant: <OSS / foreign currency / exported / none>
Owner approval: <name, time> or PENDING

EXECUTION LOG · <agenda> · <date>
| time | endpoint | Id | DocumentNumber | result (IsSuccess/ErrorCode) | before → after (re-read) |
Not done / needs approval: <list>
```

## Worked example
Illustrative numbers and IDs only. A Czech VAT-payer e-shop in CZK, bank transfer, no rounding on that payment option. The owner asks for an invoice for B2B order 10482 delivered on 2026-10-03, then to record the payment once it arrives. The accountant's rule: DUZP = delivery date.

1. Token: `POST https://identity.idoklad.cz/server/v2/connect/token`, form fields `grant_type` = `client_credentials`, `scope` = `idoklad_api`, and `application_id`, `client_id`, `client_secret` filled from `IDOKLAD_APPLICATION_ID`, `IDOKLAD_CLIENT_ID`, `IDOKLAD_CLIENT_SECRET`. Response holds `access_token`, `token_type` Bearer and `expires_in`; the agent caches it in memory and logs only "token obtained".
2. `GET /v3/VatRates` for the agenda country valid on 2026-10-03 returned `RateType` 1 at 21 and `RateType` 0 at 12. `GET /v3/Contacts?filter=(IdentificationNumber~eq~12345678)` returned PartnerId 5512.
3. `GET /v3/IssuedInvoices/Default` returned NumericSequenceId 3, DocumentSerialNumber 148, CurrencyId 1, PaymentOptionId 1, DateOfIssue 2026-10-05, DateOfMaturity 2026-10-19, one empty item.
4. POST body = the Default model with these fields changed (all other keys, including each item's remaining keys, exactly as Default returned them):
```
{
  "PartnerId": 5512,
  "Description": "Order 10482",
  "OrderNumber": "10482",
  "DateOfTaxing": "2026-10-03",
  "Items": [
    { "Name": "Product A", "Amount": 2, "Unit": "pcs", "UnitPrice": 605.00,
      "PriceType": 0, "VatRateType": 1, "VatRate": 21, "DiscountPercentage": 0 },
    { "Name": "Product B", "Amount": 2, "Unit": "pcs", "UnitPrice": 224.00,
      "PriceType": 0, "VatRateType": 0, "VatRate": 12, "DiscountPercentage": 0 }
  ]
}
```
Prices are gross (`PriceType` 0 = WithVat) as in the store. Arithmetic: A 2 × 605.00 = 1 210.00 gross; net 1 210.00 ÷ 1.21 = 1 000.00; VAT 210.00. B 2 × 224.00 = 448.00 gross; net 448.00 ÷ 1.12 = 400.00; VAT 48.00. Totals: net 1 400.00, VAT 258.00, gross 1 658.00 (1 000 + 400; 210 + 48; 1 210 + 448). `POST /v3/IssuedInvoices/Recount` returned the same `VatRateSummary` and no rounding line, so the preview shows Recount's figures. The prices were chosen to divide exactly; with other prices the cents come from Recount, not from the agent.
5. Owner approved. POST returned `IsSuccess` true, `Id` 90210, `DocumentNumber` 20260148, `PaymentStatus` 0 (Unpaid). Re-read matches; logged.
6. Payment: the bank statement shows CZK 1 658.00 on 2026-10-09 with the invoice's variable symbol. `GET /v3/IssuedDocumentPayments?filter=(InvoiceId~eq~90210)` returned none; the payment Default endpoint with `documentId` 90210 prefilled the rest. Preview approved, then:
```
POST /v3/IssuedDocumentPayments
{ "InvoiceId": 90210, "PaymentAmount": 1658.00, "PaymentOptionId": 1,
  "DateOfPayment": "2026-10-09", "SendPaymentConfirmation": false,
  "CreateIssuedTaxDocument": false }
```
Re-read: `PaymentStatus` 1 (Paid), `Prices.TotalPaid` 1 658.00, outstanding 1 658.00 − 1 658.00 = 0.00. Had CZK 1 600.00 arrived, the status would be `PartialPaid` with 58.00 outstanding, and no FullyPay.

## Quality checklist
- Credentials came from environment variables and appear nowhere in output or logs.
- Every new document started from its Default endpoint; serial number and sequence untouched.
- VAT types mapped from `VatRates` for the DUZP; totals shown from Recount and they add up.
- DUZP, issue and due dates each stated, with the accountant's DUZP rule named.
- No DELETE, no Mails call, `SendPaymentConfirmation` false, no edit of issued or exported amounts.
- Each payment has evidence, a prior check for existing payments, and approval.
- Every write checked via `IsSuccess`, `ErrorCode` and a re-read; log lists all Ids and DocumentNumbers.
- OSS, foreign-currency and exported documents flagged for the accountant, not decided.

## Sources
- iDoklad, API v3 documentation, introduction and v2 end of support: https://api.idoklad.cz/Help/v3/cs/#v2ending
- iDoklad, API v3 documentation, return values (ApiResult): https://api.idoklad.cz/Help/v3/cs/#returnValues
- iDoklad, API v3 documentation, API call limits: https://api.idoklad.cz/Help/v3/cs/#limits
- iDoklad, API v3 documentation, paging: https://api.idoklad.cz/Help/v3/cs/#paging
- iDoklad, API v3 documentation, attachments (Base64): https://api.idoklad.cz/Help/v3/cs/#attachments
- iDoklad, API v3 documentation, filtering and sorting: https://api.idoklad.cz/Help/v3/cs/#filtering
- iDoklad, API v3 documentation, HTTP codes: https://api.idoklad.cz/Help/v3/cs/#httpCodes
- iDoklad, API v3 documentation, error codes: https://api.idoklad.cz/Help/v3/cs/#errorCodes
- iDoklad, API v3 documentation, authorization and scopes: https://api.idoklad.cz/Help/v3/cs/#authorization
- iDoklad, API v3 documentation, tips (Default before POST, select): https://api.idoklad.cz/Help/v3/cs/#bestPractices
- iDoklad, API v3, Client credentials flow: https://api.idoklad.cz/Help/v3/cs/#api-ClienCredentialsFlow
- iDoklad, API v3, Authorization code flow: https://api.idoklad.cz/Help/v3/cs/#api-AuthorizationCodeFlow
- iDoklad, API v3, Refresh tokens: https://api.idoklad.cz/Help/v3/cs/#api-RefreshToken
- iDoklad, API v3, IssuedInvoices (Default, POST, Recount, list filters): https://api.idoklad.cz/Help/v3/cs/#api-IssuedInvoices
- iDoklad, API v3, CreditNotes: https://api.idoklad.cz/Help/v3/cs/#api-CreditNotes
- iDoklad, API v3, IssuedDocumentPayments: https://api.idoklad.cz/Help/v3/cs/#api-IssuedDocumentPayments
- iDoklad, API v3, ReceivedInvoices: https://api.idoklad.cz/Help/v3/cs/#api-ReceivedInvoices
- iDoklad, API v3, Contacts: https://api.idoklad.cz/Help/v3/cs/#api-Contacts
- iDoklad, API v3, NumericSequences: https://api.idoklad.cz/Help/v3/cs/#api-NumericSequences
- iDoklad, API v3, Lists (VatRates, PaymentOptions, Currencies): https://api.idoklad.cz/Help/v3/cs/#api-Lists
- iDoklad, API v3, Account (CurrentAgenda): https://api.idoklad.cz/Help/v3/cs/#api-Account
- iDoklad, API v3, Mails: https://api.idoklad.cz/Help/v3/cs/#api-Mails
- iDoklad, API v3, Reports (PDF): https://api.idoklad.cz/Help/v3/cs/#api-Reports
- iDoklad, API v3, Batch (Exported flag): https://api.idoklad.cz/Help/v3/cs/#api-Batch
- iDoklad, Nastavení – Aplikace (API keys, request count): https://www.idoklad.cz/podpora/nastaveni-aplikace-idoklad
- iDoklad, Blíží se ukončení iDoklad API v2: https://www.idoklad.cz/blog/blizi-se-ukonceni-idoklad-api-v2-zkontrolujte-sve-integrace
- iDoklad, Ceník (API requests per plan): https://www.idoklad.cz/cenik
- Zákon č. 235/2004 Sb., o dani z přidané hodnoty, § 21, § 28, § 29, § 42, § 45, § 47: https://www.zakonyprolidi.cz/cs/2004-235

## License
MIT
