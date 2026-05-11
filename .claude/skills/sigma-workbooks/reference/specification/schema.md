# Workbook Spec — Top-Level Schema

A workbook spec is a JSON representation of a Sigma workbook. This file covers the overall shape of the top-level object and the `pages` array skeleton — see the per-element and per-source reference files for the pieces that go inside.

## Top-Level Object

The object passed to `POST /v2/workbooks/spec`:

```json
{
  "name": "My Workbook",
  "folderId": "<folder-uuid>",
  "description": "Optional description",
  "schemaVersion": 1,
  "pages": [...],
  "layout": "<?xml ...?>..."
}
```

**Required:** `name`, `folderId`, `schemaVersion`, `pages`.
**Optional:** `description`, `layout`.

Use the `schemaVersion` returned by `GET /v2/workbooks/<template-id>/spec` in Step 2 of the workflow — don't hardcode it. The server will reject a spec whose `schemaVersion` doesn't match what the API expects.

## Response-Only Fields

`GET /v2/workbooks/<id>/spec` also returns these — they **must be stripped** before using the spec as a create/update body. Sending unknown top-level fields will be rejected:

- `workbookId`
- `url`
- `documentVersion`
- `latestDocumentVersion`
- `ownerId`
- `createdBy`
- `updatedBy`
- `createdAt`
- `updatedAt`

## Pages

`pages` is the core of the spec. Each page:

```json
{
  "id": "page-1",
  "name": "Overview",
  "elements": [...]
}
```

The `elements` array holds table elements, charts, KPIs, controls, and containers. See the per-element reference files.

## ID Rules

- Element IDs and column IDs must be unique within their scope.
- Use descriptive kebab-case or short random-looking IDs — both are fine. IDs are internal identifiers, not displayed to users.
- **Critical:** on `POST`, the server reassigns external IDs to internal ones. For any follow-up `PUT` (especially layout XML updates), GET the current spec first and use the IDs from the readback. Layout `elementId` references must match the current internal IDs exactly (case-sensitive).

## Minimal Working Example

The smallest spec that creates a workable workbook:

```json
{
  "name": "Sales Dashboard",
  "folderId": "<folder-uuid>",
  "schemaVersion": 1,
  "pages": [
    {
      "id": "page-1",
      "name": "Overview",
      "elements": [
        {
          "id": "sales-table",
          "kind": "table",
          "name": "Sales Data",
          "source": {
            "kind": "warehouse-table",
            "connectionId": "<conn-uuid>",
            "path": ["SALES_DB", "PUBLIC", "ORDERS"]
          },
          "columns": [
            { "id": "col-order-id", "formula": "[ORDERS/order_id]",   "name": "Order ID" },
            { "id": "col-amount",   "formula": "[ORDERS/amount]",     "name": "Amount" },
            { "id": "col-revenue",  "formula": "[ORDERS/revenue]",    "name": "Revenue" },
            { "id": "col-cost",     "formula": "[ORDERS/cost]",       "name": "Cost" },
            { "id": "col-date",     "formula": "[ORDERS/order_date]", "name": "Date" },
            { "id": "col-total",    "formula": "Sum([Amount])",       "name": "Total Amount" },
            { "id": "col-profit",   "formula": "[Revenue] - [Cost]",  "name": "Profit" }
          ]
        }
      ]
    }
  ]
}
```

Note how:
- `[ORDERS/order_id]` references a warehouse column (table prefix required).
- `Sum([Amount])` references the "Amount" column defined in the same element (no prefix).
- `[Revenue] - [Cost]` references two other columns in the same element by their `name` field.

For a realistic multi-page, multi-element spec with KPIs, charts, joins, controls, and layout, see `example-full.yaml`.
