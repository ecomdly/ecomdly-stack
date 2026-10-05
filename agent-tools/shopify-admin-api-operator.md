---
name: shopify-admin-api-operator
owner: cartlift
category: Agent tools
description: Lets an agent read and change a Shopify store through the GraphQL Admin API safely: pinned version, least-privilege scopes, cost-aware paging and bulk exports, userErrors checks, owner-approved diffs and a rollback log.
version: v1
license: MIT
updated: 2026-10-05
recommended: false
security_checked: true
url: https://ecomdly.com/skills/cartlift/shopify-admin-api-operator
raw: https://ecomdly.com/raw/cartlift/shopify-admin-api-operator.md
install: npx @ecomdly/cli add cartlift/shopify-admin-api-operator
---

# Shopify Admin API operator

Produces safe, reviewable reads and writes on one Shopify store through the GraphQL Admin API: a change plan with before/after values for the owner to approve, then an execution log that doubles as a rollback file. The naive approach (grab a token, loop REST calls, treat HTTP 200 as success) fails three ways: GraphQL answers `200 OK` even when the operation failed, throttling arrives as a `THROTTLED` error in the body rather than HTTP 429, and a "convenient" mutation like `productSet` deletes every variant you left out of the input. **The key insight: on this API, success is never the status code. It is an empty `errors` array, an empty `userErrors` list, and a re-read that shows the new value.**

## When to use
- The owner asks the agent to look something up in the store: a product by SKU, stock at a location, orders with a tag, a customer's order history, active discounts.
- The owner wants a batch change: prices, compare-at prices, tags, order notes, metafields, collection membership, inventory corrections.
- A large export (whole catalog, a year of orders) is needed as a file for another analysis.

## When not to use
- Deciding what a price should be: `competitor-price-brief`, `promotion-profit-check`, `clearance-markdown-planner` (Omnibus prior price), `free-shipping-threshold-planner`. This skill only executes a decided change.
- Marketplace channel economics: `marketplace-profitability-check`; feed quality: `product-feed-optimizer`, `merchant-center-disapproval-fixer`, `gtin-identifier-auditor`.
- Replying to customers about orders: `order-status-reply-drafter`, `warranty-claim-handler`.
- Reconciling Shopify revenue with GA4 or Ads: `revenue-discrepancy-reconciler`.
- Rewriting product copy: `product-description-writer` (then use this skill to write the approved text back).

## Inputs
| input | definition | typical source |
|---|---|---|
| Shop domain | `{shop}.myshopify.com`, not the storefront domain | Shopify admin URL |
| API version | a supported quarterly version to pin, e.g. `2026-10` | owner or integration config |
| Access token | Admin API token, read from an environment variable or secret store at run time | legacy custom app token, or client credentials grant for a Dev Dashboard app |
| Granted scopes | the scopes the token actually holds | the `scope` field returned with the token, or the app's configuration |
| Plan | Standard, Advanced, Plus or Enterprise, which sets the restore rate | Shopify admin, Settings > Plan |
| The task | what to read or change, on which records, with the target values | the owner |

Optional: location IDs for inventory, a preferred batch size, an owner-approved change window, the store currency and time zone.

Missing input → stop and ask. Never guess a shop domain, an API version or a target value, and never fall back to "typical" store settings.

## Best practices

### Access and authentication
1. **GraphQL only.** The REST Admin API is legacy as of 2024-10-01, and since 2025-04-01 new public apps must be built exclusively with the GraphQL Admin API. New fields land in GraphQL; REST answers drift out of date.
2. **One endpoint, one header.** `POST https://{shop}.myshopify.com/admin/api/{version}/graphql.json` with the token in `X-Shopify-Access-Token`. Pin the version in the URL: versions ship quarterly (1 Jan, Apr, Jul, Oct, 17:00 UTC), each stable version is supported at least 12 months, and a request to an inaccessible version silently falls forward to the oldest accessible one, so field behaviour can change under you.
3. **Know which token the store has.** Custom apps can no longer be created in the Shopify admin after 2026-01-01; those created earlier ("legacy custom apps") keep working and are managed there. New custom apps are created in the Dev Dashboard; when app and store are in the same organization the app exchanges client ID and secret at `/admin/oauth/access_token` (`grant_type=client_credentials`) for a token that expires after 24 hours (`expires_in` 86399). Cache it and refresh before expiry.
4. **Secrets never leave the secret store.** Read the token and client secret from environment variables or a secret manager. Never write them into files, logs, change plans, commits or chat; log the variable name only. A token that appears in output is compromised and must be rotated by the owner.
5. **Least privilege, per task.** Request only what the task needs: `read_products`/`write_products` (products, variants, collections), `read_inventory`/`write_inventory`, `read_orders`/`write_orders` (orders of the last 60 days), `read_customers`, `read_discounts`/`write_discounts`. Do not ask for `read_all_orders` (needs Shopify approval) unless the task really needs orders older than 60 days. Each mutation's reference page lists its scope; check it before asking the owner to grant one.
6. **Customer data is protected data.** Name, email, phone and address are Level 2 protected customer data; Shopify requires data minimisation, stated purpose, retention limits and encryption. For admin-created custom apps Level 2 access depends on the store's plan. Read only the customer fields the task needs.

### Reading efficiently
7. **Page with cursors.** Use `first` (max 250) and `after: endCursor` until `pageInfo.hasNextPage` is false. Pagination stops at 25,000 objects, so bigger sets go to bulk.
8. **Filter on the server.** Use the `query` argument: `sku:LS-01-M`, `updated_at:>'2026-09-01T00:00:00Z'`, `-tag:archive`, `tag:wholesale OR tag:b2b`, quoted phrases, `field:*` for "has a value". Fetching everything and filtering locally burns cost points.
9. **Bulk for big exports.** `bulkOperationRunQuery` returns JSONL with `__parentId` for nested rows; max five connections, two levels deep; must finish within 10 days; the result URL expires after a week. From `2026-01` an app can run up to five bulk queries per shop at once (earlier versions: one of each type) and polls `bulkOperation(id:)`; `currentBulkOperation` is deprecated.
10. **Bulk imports only after a pilot.** `bulkOperationRunMutation` takes a JSONL variables file (max 100 MB) uploaded via `stagedUploadsCreate` with `BULK_MUTATION_VARIABLES`, must finish within 24 hours, and reports errors per line in the result file. Run the same mutation on 3 to 5 records first and verify them.

### Rate limits
11. **Budget by cost, not by request count.** Objects cost 1, connections are sized by `first`/`last`, mutations cost 10, one query may not exceed 1,000 points. Restore rates: Standard 100, Advanced 200, Plus 1,000, Enterprise 2,000 points/second. Shopify does not publish bucket sizes on that page, so read `extensions.cost.throttleStatus` (`maximumAvailable`, `currentlyAvailable`, `restoreRate`) from each response and pace from that.
12. **Back off on THROTTLED.** Wait (requested cost − currently available) ÷ restore rate seconds, plus jitter, then retry the same request. Never retry a write without first checking it did not land.

### Webhooks, briefly
13. **Polling is the default; webhooks only if a receiver exists.** If the owner already runs one, verify each delivery: HMAC-SHA256 of the raw body with the app's client secret, base64, compared timing-safe with `X-Shopify-Hmac-SHA256`; reply `200` within 5 seconds. Unverified payloads are discarded.

### Writing
14. **Check three things after every mutation:** top-level `errors`, the mutation's `userErrors`, and the returned object. HTTP 200 with a non-empty `userErrors` is a failed write.
15. **Use global IDs from reads.** IDs look like `gid://shopify/ProductVariant/123`; copy them from the read, never build them from storefront URLs or SKUs.
16. **Pick the narrowest mutation.** Prices and compare-at prices: `productVariantsBulkUpdate` (one product per call, `allowPartialUpdates` default false). Product fields: `productUpdate`. `productSet` replaces list fields: variants, collections and metafields missing from the input are deleted, so use it only for full syncs the owner approved.
17. **Inventory through quantities, with a reason.** `inventorySetQuantities` (absolute, `available` or `on_hand`) or `inventoryAdjustQuantities` (delta), with a `reason` such as `correction` or `cycle_count_available`, a `referenceDocumentUri` (GID format preferred) and `changeFromQuantity` so a stale read fails with `CHANGE_FROM_QUANTITY_STALE` instead of overwriting a sale. From `2026-04` these mutations require `@idempotent(key: ...)`; reuse the key on retry so a timeout does not apply the change twice.
18. **Tags and notes.** `tagsAdd` appends tags to an order, product, customer or discount; `orderUpdate` with `tags` replaces all existing tags. Notes via `orderUpdate` overwrite the old note, so log the old text.
19. **Metafields atomically.** `metafieldsSet` takes up to 25 per call, is all-or-nothing, and accepts `compareDigest` to refuse a write if someone changed the value since the read.

## Process
1. **Confirm setup**: shop domain, pinned version, token present in the environment (check presence, never print it), granted scopes cover the task.
2. **Read**: query the target records with only the needed fields; save IDs and current values.
3. **Plan**: build the change plan (Output format) with before, after and difference per record, scope used, batch size, and the mutation per batch.
4. **Approve**: show the plan to the owner and wait for an explicit yes. A changed plan needs a new yes.
5. **Pilot**: execute the first small batch (up to 5 records or one product).
6. **Verify**: check `errors`, `userErrors`, returned values, then re-read and compare with the plan.
7. **Continue** in batches, pacing by `throttleStatus`; stop the run at the first unexpected `userErrors` and report.
8. **Log**: write the execution log with before/after values, timestamps and any idempotency keys; the before column is the rollback plan.
9. **Run the quality checklist.**

## Pitfalls and edge cases
- **Money is a string in shop currency.** Send `"26.90"`, not 26.9; multi-currency markets may convert or override it, so check market price lists before promising a price abroad.
- **Compare-at price is a sale claim.** Setting it creates a visible strike-through; in the EU the reference must follow the Omnibus prior-price rule, so route that decision to `clearance-markdown-planner`.
- **One product per `productVariantsBulkUpdate` call.** Variants of different products need separate calls.
- **Orders beyond 60 days** are invisible without `read_all_orders`; an empty result is not "no orders".
- **Search index lag.** A record changed seconds ago may not match a `query` filter yet; verify by ID.
- **Inventory by location.** Quantities live per location and inventory item; a variant-level total hides which location changed.
- **Fulfilment writes notify people.** `fulfillmentCreate` has a `notifyCustomer` option; marking orders fulfilled is a customer-facing action, not a data fix.
- **Timeouts on writes.** A network timeout does not mean failure; re-read before retrying.

## Rules
- Never write without an owner-approved change plan. Reads need no approval.
- Never, without a separate explicit approval naming the records: deletes (products, variants, metafields, discounts), refunds, order cancellations, fulfilments that notify customers, `productSet` full syncs, customer data exports, or any scope beyond the task.
- Never print, log or store the access token or client secret; never send store data to any service other than the store's own Admin API.
- Never invent IDs, values or results; report what the API returned.
- Stop on the first unexpected `userErrors` and report instead of working around it.
- Store prices, discounts and stock decisions belong to the owner; the agent executes them, it does not choose them.

## Output format
```
CHANGE PLAN · <shop>.myshopify.com · API <version> · prepared <date time, tz>
Task: <one line> · scopes used: <list> · records: <n> · batches: <n> × <size>
| # | record (GID) | handle / SKU | field | before | after | difference |
Mutation per batch: <name> · checks: errors, userErrors, re-read
Not included / needs separate approval: <list or none>
Owner approval: <name, time> or PENDING

EXECUTION LOG · <shop> · API <version> · started <time> · finished <time>
| batch | record (GID) | field | before | after (read back) | status | userErrors | idempotency key |
Cost: requested <n> · actual <n> · lowest currentlyAvailable <n> · throttled retries <n>
Rollback: re-run <mutation> with the "before" column for rows with status OK
Open issues: <list or none>
```

## Worked example
Illustrative numbers and IDs only. Slovak store in EUR on the Standard plan; the owner decided to raise the linen shirt prices and asked the agent to apply them.

Read (pinned `2026-10`):
```
query VariantsBySku($q: String!) {
  productVariants(first: 10, query: $q) {
    nodes { id sku displayName price compareAtPrice product { id } }
    pageInfo { hasNextPage endCursor }
  }
}
```
Variables: `{"q": "sku:LS-01-S OR sku:LS-01-M OR sku:LS-01-L"}` returned three variants of `gid://shopify/Product/8123456789`, no compare-at prices, `hasNextPage` false.

Change plan shown to the owner and approved:
```
| 1 | gid://shopify/ProductVariant/45100000001 | LS-01-S | price | 24.90 | 26.90 | +2.00 (+8.0%) |
| 2 | gid://shopify/ProductVariant/45100000002 | LS-01-M | price | 24.90 | 26.90 | +2.00 (+8.0%) |
| 3 | gid://shopify/ProductVariant/45100000003 | LS-01-L | price | 29.90 | 32.90 | +3.00 (+10.0%) |
```
Mutation (one product, one batch; default mutation cost is 10 points):
```
mutation UpdateVariantPrices($productId: ID!, $variants: [ProductVariantsBulkInput!]!) {
  productVariantsBulkUpdate(productId: $productId, variants: $variants) {
    productVariants { id sku price compareAtPrice }
    userErrors { field message }
  }
}
```
Variables: `{"productId": "gid://shopify/Product/8123456789", "variants": [{"id": "gid://shopify/ProductVariant/45100000001", "price": "26.90"}, {"id": "gid://shopify/ProductVariant/45100000002", "price": "26.90"}, {"id": "gid://shopify/ProductVariant/45100000003", "price": "32.90"}]}`

Check: response has no `errors`, `userErrors` is `[]`, and `productVariants` returns 26.90, 26.90, 32.90. If `userErrors` were non-empty, nothing is retried: with `allowPartialUpdates` false the agent re-reads the three variants, logs what is actually stored and reports the message to the owner. Arithmetic: 2.00 ÷ 24.90 = 8.0%; 3.00 ÷ 29.90 = 10.0%. Rollback is the same mutation with 24.90, 24.90, 29.90.

## Quality checklist
- Version pinned in the URL and stated in the plan and log.
- Token read from the environment; it appears nowhere in output.
- Scopes are the minimum for the task; no `read_all_orders` without a stated need.
- Every write was in an approved plan; destructive or customer-facing actions had their own approval.
- Every mutation result checked for `errors` and `userErrors`, and re-read.
- Before values logged for every changed field; rollback is executable from the log.
- Inventory writes carry reason, reference, `changeFromQuantity` and an idempotency key.
- Pagination ran to `hasNextPage` false, or bulk was used above 25,000 objects.

## Sources
- Shopify, GraphQL Admin API reference: https://shopify.dev/docs/api/admin-graphql
- Shopify, REST Admin API reference (legacy status, 2025-04-01 rule): https://shopify.dev/docs/api/admin-rest
- Shopify, API versioning: https://shopify.dev/docs/api/usage/versioning
- Shopify, GraphQL Admin API rate limits: https://shopify.dev/docs/apps/build/apis/graphql-admin/rate-limits
- Shopify, API limits: https://shopify.dev/docs/api/usage/limits
- Shopify, Response status and error codes: https://shopify.dev/docs/api/usage/response-codes
- Shopify Help Center, Custom apps: https://help.shopify.com/en/manual/apps/app-types/custom-apps
- Shopify, Client credentials grant: https://shopify.dev/docs/apps/build/authentication-authorization/client-credentials-grant
- Shopify, Get API access tokens for Dev Dashboard apps: https://shopify.dev/docs/apps/build/dev-dashboard/get-api-access-tokens
- Shopify, Access scopes: https://shopify.dev/docs/api/usage/access-scopes
- Shopify, Protected customer data: https://shopify.dev/docs/apps/launch/protected-customer-data
- Shopify, Paginating results with GraphQL: https://shopify.dev/docs/api/usage/pagination-graphql
- Shopify, Search syntax: https://shopify.dev/docs/api/usage/search-syntax
- Shopify, Bulk operations, queries: https://shopify.dev/docs/api/usage/bulk-operations/queries
- Shopify, Bulk operations, imports: https://shopify.dev/docs/api/usage/bulk-operations/imports
- Shopify, Global IDs: https://shopify.dev/docs/api/usage/gids
- Shopify, productVariantsBulkUpdate: https://shopify.dev/docs/api/admin-graphql/latest/mutations/productVariantsBulkUpdate
- Shopify, productSet: https://shopify.dev/docs/api/admin-graphql/latest/mutations/productSet
- Shopify, productVariants query: https://shopify.dev/docs/api/admin-graphql/latest/queries/productVariants
- Shopify, inventorySetQuantities: https://shopify.dev/docs/api/admin-graphql/latest/mutations/inventorySetQuantities
- Shopify, inventoryAdjustQuantities: https://shopify.dev/docs/api/admin-graphql/latest/mutations/inventoryAdjustQuantities
- Shopify, Manage inventory quantities and states: https://shopify.dev/docs/apps/build/orders-fulfillment/inventory-management-apps/manage-quantities-states
- Shopify, Changelog, Making idempotency mandatory for inventory adjustments and refund mutations: https://shopify.dev/changelog/making-idempotency-mandatory-for-inventory-adjustments-and-refund-mutations
- Shopify, metafieldsSet: https://shopify.dev/docs/api/admin-graphql/latest/mutations/metafieldsSet
- Shopify, tagsAdd: https://shopify.dev/docs/api/admin-graphql/latest/mutations/tagsAdd
- Shopify, orderUpdate: https://shopify.dev/docs/api/admin-graphql/latest/mutations/orderUpdate
- Shopify, fulfillmentCreate: https://shopify.dev/docs/api/admin-graphql/latest/mutations/fulfillmentCreate
- Shopify, HTTPS webhook delivery (HMAC verification): https://shopify.dev/docs/apps/build/webhooks/subscribe/https

## License
MIT
