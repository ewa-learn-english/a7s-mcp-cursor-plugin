# A7S Skipper — Cursor Plugin

Cursor IDE plugin for [A7S Skipper](https://bi.e2w2a.pro) — a BI platform for building YAML-driven dashboards on ClickHouse with a semantic metrics layer.

## What's included

| Component | Description |
|-----------|-------------|
| **MCP Server** | Remote connection to A7S Skipper via OAuth 2.1 (PKCE) |
| **8 Dashboard Rules** | YAML authoring reference for datasources, charts, tables, filters, mappings, markdown |
| **Dashboard Authoring Skill** | Guided workflow: DISCOVER → EXPLORE → BUILD → VALIDATE |

## Setup

1. Install the plugin in Cursor (Team Marketplace or local install)
2. The MCP server appears in Settings → MCP Servers as **a7s-skipper**
3. On first tool call, Cursor will open the OAuth login flow
4. After consent, all MCP tools are available

## MCP Tools

Once connected, the AI agent has access to:

**Semantic Layer** — `list_metrics`, `search_catalog`, `explore_metric`

**Data Exploration** — `preview_query`, `get_column_stats`, `validate_sql`

**Dashboard CRUD** — `create_dashboard`, `create_tab`, `create_section`, `update_section`, `update_shared`, `delete_dashboard`

**Validation** — `validate_dashboard`, `validate_sql`

**Resources** — `skipper://dashboards`, `skipper://design-system`, `skipper://query-results/{id}`

## Example prompts

- "Create a marketing dashboard with daily spend and ROI from `marts.marketing_creatives`"
- "Add a cohort analysis tab to the existing marketing dashboard"
- "What metrics are available in the semantic layer for revenue?"
- "Show me column stats for `marts.orders` to help build filters"
- "Validate the SQL for my datasource query"

## Rules reference

The plugin includes 8 rules covering every aspect of dashboard YAML:

- **dashboard-yaml-structure** — Top-level structure, sections, tabs, grid layout
- **dashboard-builder-guide** — Style conventions, patterns, checklists
- **dashboard-datasources** — ClickHouse sources, projections, views, skills, filter placeholders
- **dashboard-chart** — Chart presets, series, tooltips, multi-track, ECharts options
- **dashboard-rich-table** — Columns, grouping, pivot, filtering, conditional formatting
- **dashboard-filters** — Filter types, sources, hierarchy, URL serialization
- **dashboard-mappings** — Value mapping dictionaries
- **dashboard-markdown** — Dynamic text widgets with Handlebars templates

## Development

This plugin is maintained as a `cursor-plugin/` subdirectory in the [a7s monorepo](https://github.com/ewa-learn-english/a7s) and published to a standalone repo via `git subtree push`.

```bash
# Sync rules from packages/cli/rules/
task plugin:sync

# Push to the plugin repository
task plugin:push
```

## License

Proprietary — EWA
