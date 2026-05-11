# Tables

The `table` element displays data in tabular format — the most common element kind and the primary way data enters a workbook. Charts, KPIs, and other elements usually point their `source` at a table.

## Basic Shape

```json
{
  "id": "sales-table",
  "kind": "table",
  "name": "Sales Data",
  "source": {
    "kind": "warehouse-table",
    "connectionId": "<conn-uuid>",
    "path": ["DATABASE", "SCHEMA", "TABLE"]
  },
  "columns": [
    { "id": "col-1", "formula": "[TABLE/column_name]", "name": "Column Name" },
    { "id": "col-2", "formula": "Sum([Column Name])",   "name": "Total" }
  ],
  "order": ["col-1", "col-2"]
}
```

See `sources.md` for all source kinds (warehouse-table, join, data-model, etc.) and `formulas.md` for the column reference rules.

## Optional Fields

### `order`
Array of column IDs controlling left-to-right display order. If omitted, columns appear in declaration order.

```json
"order": ["col-order-id", "col-amount", "col-profit"]
```

### `groupings`
Pivot / aggregation views. Each grouping has an `id`, a `groupBy` list of column IDs, and a `calculations` list of column IDs.

```json
"groupings": [
  {
    "id": "by-region",
    "groupBy": ["col-region"],
    "calculations": ["col-total", "col-profit"]
  }
]
```

### `filters`
Element-level row filters. Top-N is the most common variant:

```json
"filters": [
  {
    "id": "top-20",
    "columnId": "col-revenue",
    "kind": "top-n",
    "rankingFunction": "rank",
    "mode": "top-n",
    "rowCount": 20,
    "includeNulls": "when-no-value-is-selected"
  }
]
```

> **`rowCount` takes a number literal only** — it cannot be parametrized by a control. `"rowCount": "[TopN]"` or any other control-reference shape is rejected. Control bindings apply to **filter values**, not to structural fields like `rowCount`, `rankingFunction`, `mode`, or `kind`. Same goes for the equivalent `filters` block on charts (see `charts.md`). To vary the cap interactively today, duplicate the element per cap.

## Columns

Every column needs `id`, `name`, and `formula`. Optional: `format` (see `formatting.md`).

```json
{
  "id": "col-revenue",
  "name": "Revenue",
  "formula": "[ORDERS/revenue]",
  "format": { "kind": "number", "formatString": "$,.0f" }
}
```

**Formula rules** — see `formulas.md`. The single most common mistake is a bare `[col]` reference when a source prefix is required.

---

# Pivot Tables

The `pivot-table` element is a sibling of `table` for cross-tab analysis — measure cells aggregated across one or more row/column dimensions. Verified end-to-end on staging.

## Shape

```json
{
  "id": "deployments-pivot",
  "kind": "pivot-table",
  "name": "Deployments by cloud and env",
  "source": { "kind": "table", "elementId": "deployments-source" },
  "columns": [
    { "id": "piv-cloud", "name": "Cloud",       "formula": "[Deployments/Cloud]" },
    { "id": "piv-env",   "name": "Environment", "formula": "[Deployments/Environment]" },
    { "id": "piv-count", "name": "Deployments", "formula": "CountDistinct([Deployments/Deployment UUID])",
      "format": { "kind": "number", "formatString": ",.0f" } }
  ],
  "values": ["piv-count"]
}
```

| Field | Required | Notes |
|---|---|---|
| `id` | yes | Unique on the page |
| `kind` | yes | Always `"pivot-table"` |
| `source` | yes | Same source kinds as `table` — warehouse-table, cross-element table reference, sql, data-model |
| `columns` | yes | Same shape as a `table`'s `columns` (id + formula + name + format) |
| `values` | yes | Array of column IDs that act as the **measures** (the cells of the pivot). The remaining columns act as row/column dimensions. |
| `name` | no | Display title — accepted even though not in the OpenAPI required list |

## Round-Trip Quirk: Column Reordering

Sigma reorders the `columns` array on round-trip — **value columns first, then dimensions** — regardless of the order you submit. If you GET → edit → PUT, expect a non-substantive diff in the columns array. The `values` array preserves the IDs you sent, so the rendered output is unchanged.

## What `values` vs. non-`values` columns mean

The OpenAPI surfaces only `columns` and `values`; there is no separate `rows`/`pivotRows`/`pivotColumns` field. Sigma infers row vs. column dimensions from the non-`values` columns automatically. To control the layout further (which dimension goes on rows vs. on the top header), today you'll need to use the UI rather than the spec.
