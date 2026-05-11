# Sources (non-warehouse)

Every element with columns has a `source` that defines where its data comes from. This file covers source kinds **other than** `warehouse-table`.

**For warehouse-table** — the most common source — see the dedicated `sources-warehouse.md`. It's loaded by default because nearly every workbook uses it.

**For discovery** (finding connection IDs, paths, data model IDs, element IDs) see `source-discovery.md`.

---

## table (cross-element reference)

Sources another element in the same workbook. This is the most common source kind for charts, KPIs, and derived tables.

```json
{
  "kind": "table",
  "elementId": "<id of another element on some page>"
}
```

Column formulas reference that element's columns using its `name`:
- Element named "Sales Table" → `[Sales Table/Revenue]`

## data-model

References an element from an existing data model.

```json
{
  "kind": "data-model",
  "dataModelId": "<data-model-uuid>",
  "elementId": "<element-uuid within that model>"
}
```

## join

Joins multiple warehouse tables into one logical source. Specify a `primarySource` and an array of `joins`. Each join has `left`, `right`, `columns` (the join keys), `name` (used as the prefix in column formulas — `[<name>/column]`), and `joinType`.

```json
{
  "kind": "join",
  "name": "Sales Star",
  "primarySource": {
    "kind": "warehouse-table",
    "connectionId": "<conn-uuid>",
    "path": ["DB", "SCHEMA", "F_POS"]
  },
  "joins": [
    {
      "name": "Sales",
      "joinType": "left-outer",
      "left":  { "kind": "warehouse-table", "connectionId": "<conn-uuid>", "path": ["DB", "SCHEMA", "F_POS"] },
      "right": { "kind": "warehouse-table", "connectionId": "<conn-uuid>", "path": ["DB", "SCHEMA", "F_SALES"] },
      "columns": [{ "left": "[Order Number]", "right": "[Order Number]" }]
    }
  ]
}
```

`joinType` values: `"inner"`, `"left-outer"`, `"right-outer"`, `"full-outer"`, `"cross"`.

**Column formula prefixes with joins** — see `formulas.md` for the full rules:
- Primary-source columns use the **join's top-level `name`**: `[Sales Star/Order Number]`
- Joined-table columns use the **join leg's `name`**: `[Sales/Cust Key]`
- Warehouse path segments are **not** used as prefixes inside a join.

## Other Source Kinds

These exist but are less common; model the shape off an existing workbook's spec via `GET /v2/workbooks/<id>/spec`:

- `"sql"` — custom SQL query
- `"union"` — unions multiple elements row-wise
- `"transpose"` — transposes rows/columns
