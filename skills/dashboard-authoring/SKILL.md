# A7S Skipper — Dashboard Authoring Skill

Use this skill when the user wants to create, modify, or explore BI dashboards powered by A7S Skipper. This skill orchestrates MCP tools for data exploration, semantic catalog, and dashboard CRUD.

## When to activate

- User asks to create a dashboard, tab, section, or widget
- User asks to explore data, metrics, or columns in ClickHouse
- User asks to build or modify YAML dashboard files
- User mentions "A7S", "Skipper", "dashboard", "datasource", "metric", "semantic layer"
- User provides a ClickHouse table or SQL query and wants it visualized

## Prerequisites

The `a7s-skipper` MCP server must be connected. If the user hasn't authenticated yet, guide them to trigger the OAuth flow by calling any MCP tool (the IDE will prompt for login automatically).

## Workflow: DISCOVER → EXPLORE → BUILD → VALIDATE

Follow this sequence strictly. Never skip steps.

### Phase 1: DISCOVER

**Goal:** Understand what data and metrics are available.

1. **Check the semantic layer first:**
   - Call `list_metrics` to see all available semantic metrics
   - Call `search_catalog` with keywords to find relevant metrics
   - Semantic metrics are pre-defined, validated, and preferred over raw SQL

2. **Explore raw tables if needed:**
   - Call `preview_query` with `SELECT * FROM schema.table LIMIT 5` to discover exact column names and types
   - **NEVER guess column names** — always verify with `preview_query` or `validate_sql`

3. **Study existing dashboards:**
   - Read `skipper://dashboards` resource to see existing dashboard structures
   - Study their datasource SQL, filter patterns, widget structure, and naming conventions
   - Reuse patterns — consistency matters more than novelty

### Phase 2: EXPLORE

**Goal:** Understand data shape, distributions, and edge cases.

1. Call `preview_query` to sample data and verify columns
2. Call `get_column_stats` to understand value distributions before building filters
3. Call `explore_metric` for semantic metrics to see their SQL and dimensions
4. Pay attention to `column_types` in responses — use only listed column names
5. Large results (>30 rows) are truncated. Full data is available via `skipper://query-results/{id}` resource link

### Phase 3: BUILD

**Goal:** Create or modify the dashboard using MCP CRUD tools.

Available tools and their purpose:

| Tool | Purpose |
|------|---------|
| `create_dashboard` | Create dashboard folder with `+layout.yaml` and first tab |
| `create_tab` | Add a tab to an existing dashboard |
| `create_section` | Add a content section/widgets to a tab |
| `update_section` | Modify an existing section |
| `update_shared` | Modify shared datasources/filters/mappings |
| `delete_dashboard` | Remove a dashboard and all its content |

**Critical rules:**
- Always follow "Next steps" from the previous tool response
- Reference shared datasources via `{{source:name}}` — never hardcode table names in widgets
- Use filter macros (`${filters.date_range('toDate(col)')}`) — never hardcode date ranges
- Base source SQL uses **ClickHouse** syntax; projection/view SQL uses **DuckDB** syntax

### Phase 4: VALIDATE

**Goal:** Verify everything works before finalizing.

1. Call `validate_sql` to verify every datasource SQL (checks columns, types, estimated rows)
2. Call `validate_dashboard` to confirm YAML schema compliance
3. Fix any errors reported — do not leave broken SQL or invalid YAML

## Naming Conventions

| Element | Style | Example |
|---------|-------|---------|
| Dashboard name | Human-readable, 2-4 words | "Marketing Performance" |
| Dashboard ID | snake_case | `marketing_performance` |
| Tab title | 1-3 words | "Overview", "By Channel" |
| Section ID | snake_case | `key_metrics`, `trend_analysis` |
| Widget ID | kebab-case | `daily-costs-table`, `revenue-chart` |
| Datasource key | snake_case | `daily_marketing`, `by_media_source` |
| All user-facing titles | Human-readable, properly capitalized | Never technical slugs |

## SQL Conventions

```sql
-- Leading-comma alignment
SELECT toDate(install_date) AS date
     , SUM(costs) AS costs
     , SUM(revenue_net) AS revenue_net
FROM marts.marketing_creatives
WHERE ${filters.date_range('toDate(install_date)')}
GROUP BY date
ORDER BY date DESC
```

- Leading commas for column list (easy to add/remove)
- `${filters.<id>('column_expr')}` for ClickHouse-side filtering
- `GROUP BY` by column name, not ordinal
- `ORDER BY` primary metric `DESC` (or date `DESC` for time-series)
- No trailing semicolons
- Alias all expressions (`SUM(costs) AS costs`)
- Always guard division: `NULLIF(denominator, 0)`

## Dashboard Structure Patterns

### Drill-down tab hierarchy (recommended)

Structure tabs from high-level to granular:

1. **Overview** — daily aggregated, no grouping
2. **By dimension** — grouped by one dimension (media source, language)
3. **By multiple** — grouped by 2-3 dimensions
4. **Charts** — loss analysis, cohort, YoY comparison

### Widget types

| Type | When to use |
|------|-------------|
| `rich_table` | Tabular data with sorting, filtering, grouping |
| `chart` | Time series, comparisons, distributions |
| `markdown` | KPI summaries, dynamic text, navigation links |

### Chart presets

| Preset | Best for |
|--------|----------|
| `trend` | Time series (line/bar over dates) |
| `comparison` | Year-over-year comparisons |
| `breakdown` | Category splits (stacked/grouped bars) |
| `loss_analysis` | Profit/loss horizontal bars |
| `cohort` | Cohort retention/growth curves |
| `multi_track` | Multiple stacked panels sharing X-axis |

## Quality Checklist

Before delivering a dashboard, verify:

- [ ] Every datasource has `ai_hint` explaining the data shape
- [ ] Every section has `ai_hint` for AI assistant context
- [ ] Dashboard has rich `ai` metadata block (name, description, dimensions, metrics, examples)
- [ ] Filter macros used (no hardcoded dates)
- [ ] `{{source:key}}` references used (no hardcoded DuckDB table names)
- [ ] Rich tables have: `column_order`, full column definitions, `zebra: true`, sorting, export
- [ ] Charts use appropriate presets
- [ ] SQL validated via `validate_sql`
- [ ] Dashboard validated via `validate_dashboard`
- [ ] Design-system palettes used (`default`, `ocean`, `sunset`) — not arbitrary hex colors

## Design System

- Use canonical chart palette names: `default`, `ocean`, `sunset`
- Prefer named design-system colors over custom hex values
- Keep semantic colors meaningful: success/warning/error/info are for status indicators only
- If available, inspect `skipper://design-system` and `skipper://design-system/palettes` MCP resources

## Error Handling

- If `preview_query` fails, check table/schema name and try `SHOW TABLES` or `DESCRIBE TABLE`
- If `validate_sql` reports column errors, re-run `preview_query` to verify actual column names
- If `validate_dashboard` fails, read the error message carefully — it points to exact YAML paths
- If a tool returns an error about permissions, the user may need additional RBAC grants

## Tips

- Start with `list_metrics` — the semantic layer has curated, tested metrics
- Use `get_column_stats` before creating select filters — it shows all unique values
- For wide tables, use `col_span: 15` to enable horizontal scroll
- For multi-track charts, use `row_span: 2` or higher (300px per track)
- Inline widget SQL (`datasource: { sql: ... }`) is great for one-off calculations
- View datasources (`type: view`) are perfect for joining two ClickHouse sources locally in DuckDB
