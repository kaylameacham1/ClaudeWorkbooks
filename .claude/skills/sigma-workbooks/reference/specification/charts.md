# Charts

Chart elements: `line-chart`, `bar-chart`, `donut`. They share the same skeleton — a `source`, a `columns` array, and axis/value pointers that reference column IDs.

## Common Fields

| Field | Required | Notes |
|---|---|---|
| `kind` | yes | `"line-chart"`, `"bar-chart"`, or `"donut-chart"` |
| `id` | yes | Unique on the page |
| `name` | yes | Display title |
| `source` | yes | Usually `{ "kind": "table", "elementId": "<source-table-id>" }` — see `sources.md` |
| `columns` | yes | Inline column definitions for the chart's own columns |

Every column in `columns` gets an `id`, a `formula`, a `name`, and optional `format` (see `formatting.md`).

Remember: formulas on a chart that sources another element must use the source's prefix (`[<SourceName>/col]`). See `formulas.md`.

---

## Line Chart

```json
{
  "id": "sales-over-time",
  "kind": "line-chart",
  "name": "Sales over time",
  "source": { "kind": "table", "elementId": "sales-table" },
  "columns": [
    { "id": "col-month", "formula": "DateTrunc(\"month\", [Master/Date])", "name": "Month",
      "format": { "kind": "datetime", "formatString": "%b %Y" } },
    { "id": "col-sales", "formula": "Sum([Master/Sales Amount])", "name": "Sales",
      "format": { "kind": "number", "formatString": "$,.0f" } }
  ],
  "xAxis": { "id": "col-month" },
  "yAxis": [{ "id": "col-sales" }]
}
```

- `xAxis` — single `{ id, sort? }`
- `yAxis` — array of `{ id }` (multiple series)
- `xAxis.sort` shape: `{ "by": "<colId>", "direction": "ascending" | "descending" }`

## Bar Chart

Same axis shape as line-chart. Adds `stacking`.

```json
{
  "id": "sales-by-region",
  "kind": "bar-chart",
  "name": "Sales by region",
  "source": { "kind": "table", "elementId": "sales-table" },
  "columns": [
    { "id": "col-region", "formula": "[Master/Store Region]", "name": "Region" },
    { "id": "col-sales",  "formula": "Sum([Master/Sales Amount])", "name": "Sales",
      "format": { "kind": "number", "formatString": "$,.0f" } }
  ],
  "xAxis": { "id": "col-region" },
  "yAxis": [{ "id": "col-sales" }],
  "stacking": "none"
}
```

`stacking`: `"none"` | `"stacked"` | `"100"`

Add a sort to put categories in descending order of a measure:

```json
"xAxis": {
  "id": "col-region",
  "sort": { "by": "col-sales", "direction": "descending" }
}
```

## Donut

Uses `value` and `color` instead of `xAxis` / `yAxis`.

```json
{
  "id": "sales-by-family",
  "kind": "donut-chart",
  "name": "Sales by product family",
  "source": { "kind": "table", "elementId": "sales-table" },
  "columns": [
    { "id": "col-family", "formula": "[Master/Product Family]", "name": "Family" },
    { "id": "col-sales",  "formula": "Sum([Master/Sales Amount])", "name": "Sales",
      "format": { "kind": "number", "formatString": "$,.0f" } }
  ],
  "value": { "id": "col-sales" },
  "color": { "id": "col-family",
             "sort": { "by": "col-sales", "direction": "descending" } }
}
```

`holeValue` is optional. When set, it references one of the donut's columns by ID — that column's aggregated value drives the hole label/render — not a literal number. Shape:

```json
"holeValue": { "id": "col-sales" }
```

## Element-Level Filters (Top-N, etc.)

Charts take the same `filters` array as tables — the top-N example in `tables.md` applies to `bar-chart`, `line-chart`, and `donut-chart` without changes. Use this to cap a chart to the top N categories by some measure.

Top 10 regions by `Sales` on a bar chart:

```json
{
  "id": "top-regions",
  "kind": "bar-chart",
  "name": "Top 10 regions",
  "source": { "kind": "table", "elementId": "sales-table" },
  "columns": [
    { "id": "col-region", "formula": "[Master/Store Region]", "name": "Region" },
    { "id": "col-sales",  "formula": "Sum([Master/Sales Amount])", "name": "Sales",
      "format": { "kind": "number", "formatString": "$,.0f" } }
  ],
  "xAxis": { "id": "col-region", "sort": { "by": "col-sales", "direction": "descending" } },
  "yAxis": [{ "id": "col-sales" }],
  "stacking": "none",
  "filters": [
    {
      "id": "top-10",
      "columnId": "col-sales",
      "kind": "top-n",
      "rankingFunction": "rank",
      "mode": "top-n",
      "rowCount": 10,
      "includeNulls": "when-no-value-is-selected"
    }
  ]
}
```

`rowCount` takes a number literal — it cannot be bound to a control (see `controls.md`, "Where Control Bindings Apply").

## Known Unsupported Features

- No explicit `color` / breakdown channel on `bar-chart` / `line-chart` — both elements encode color implicitly from the data (one color per `yAxis` series). To produce a stacked-by-category bar chart, you'd need one `yAxis` series per known category value.
- No delta / comparison field on `kpi-chart` (see `kpis.md`). To show a comparison, stack two `kpi-chart` elements side-by-side via `layout.md` or use a chart.

## Other Chart Kinds

These are all valid `kind` values per the OpenAPI; documented examples for the most common are above. The shape mirrors the `bar-chart`/`line-chart` pattern (`source`, `columns`, `xAxis`, `yAxis`):

- `area-chart`, `combo-chart`, `scatter-chart` — same shape as `bar-chart`/`line-chart`, just a different `kind`.
- `pie-chart` — same shape as `donut-chart` (`value` + `color`).
- `pivot-table` — uses `values` instead of `yAxis`; useful for cross-tab analysis.

For element-level reference of `kind: "text"` (free-form Markdown blocks), see `text.md`.
