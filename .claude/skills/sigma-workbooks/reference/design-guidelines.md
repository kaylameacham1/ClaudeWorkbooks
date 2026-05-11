# Dashboard Design Guidelines

These guidelines apply to ALL Sigma workbooks built with this skill. Follow them exactly.

## Page Structure

Every dashboard has exactly two pages:

1. **Page 1** — named for the dashboard content (e.g., "Executive Overview"). Visible.
2. **Page 2** — always named **"Data"**. Hidden (`"hidden": true` on the page object). Contains the master table element(s) that page 1 sources from.

All charts, KPIs, and controls on page 1 source from the master table on the Data page via cross-page `elementId` references. This pattern is confirmed to work — see `example-full.yaml` for a real example.

## Controls Row

- All filter controls go in a single `kind: "container"` element at the top of the dashboard.
- Lay out controls in **one horizontal row** using a `GridContainer` in the layout XML.
- Divide 24 columns evenly: 3 controls = 8 cols each, 2 controls = 12 cols each, 4 controls = 6 cols each.
- Place the controls container in rows `1 / 4` of the page grid.

## KPI Row

- KPIs appear directly below the controls row (rows `4 / 14`).
- All KPIs must be inside a `kind: "container"` element with **rounded edges**.
  - Add `"style": { "borderRadius": "8px" }` to the container element JSON.
- Divide 24 columns evenly: 4 KPIs = 6 cols each, 3 KPIs = 8 cols each.

## Charts Section

- Charts appear below the KPI row (starting at row 14).
- Full-width trend/line charts span all 24 columns, rows `14 / 28`.
- Side-by-side charts each take 12 columns, rows `28 / 42`.

## Standard Layout Template

Replace `{...}` placeholders with actual element IDs from the GET response after create.

```xml
<?xml version="1.0" encoding="utf-8"?>
<Page type="grid" gridTemplateColumns="repeat(24, 1fr)" gridTemplateRows="auto" id="{overview-page-id}">
  <GridContainer elementId="{ctrl-container-id}" type="grid" gridColumn="1 / 25" gridRow="1 / 4"
                 gridTemplateColumns="repeat(24, 1fr)" gridTemplateRows="auto">
    <LayoutElement elementId="{ctrl-1-id}" gridColumn="1 / 9"  gridRow="1 / 4"/>
    <LayoutElement elementId="{ctrl-2-id}" gridColumn="9 / 17" gridRow="1 / 4"/>
    <LayoutElement elementId="{ctrl-3-id}" gridColumn="17 / 25" gridRow="1 / 4"/>
  </GridContainer>
  <GridContainer elementId="{kpi-container-id}" type="grid" gridColumn="1 / 25" gridRow="4 / 14"
                 gridTemplateColumns="repeat(24, 1fr)" gridTemplateRows="auto">
    <LayoutElement elementId="{kpi-1-id}" gridColumn="1 / 7"  gridRow="1 / 10"/>
    <LayoutElement elementId="{kpi-2-id}" gridColumn="7 / 13" gridRow="1 / 10"/>
    <LayoutElement elementId="{kpi-3-id}" gridColumn="13 / 19" gridRow="1 / 10"/>
    <LayoutElement elementId="{kpi-4-id}" gridColumn="19 / 25" gridRow="1 / 10"/>
  </GridContainer>
  <LayoutElement elementId="{trend-chart-id}"  gridColumn="1 / 25" gridRow="14 / 28"/>
  <LayoutElement elementId="{left-chart-id}"   gridColumn="1 / 13" gridRow="28 / 42"/>
  <LayoutElement elementId="{right-chart-id}"  gridColumn="13 / 25" gridRow="28 / 42"/>
</Page>
<Page type="grid" gridTemplateColumns="repeat(24, 1fr)" gridTemplateRows="auto" id="{data-page-id}">
  <LayoutElement elementId="{master-table-id}" gridColumn="1 / 25" gridRow="1 / 21"/>
</Page>
```

## Build Workflow (Required Order)

1. **POST** the spec without `layout` — IDs get remapped on create.
2. **GET** the created spec (`Accept: application/json`) to capture real element IDs.
3. **PUT** the spec back with the `layout` field populated using real IDs from step 2.

Never include layout in the initial POST — `elementId` references in layout XML must match the server-assigned IDs, not the ones you wrote in the spec.
