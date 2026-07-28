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
4. Continue with Google, review the requested scopes, and select **Allow**
5. Cursor receives the OAuth callback and loads the MCP tools

The OAuth client is public and uses PKCE. The client ID in `.mcp.json` is not a secret; a
`CLIENT_SECRET` must never be added to the plugin.

## Updating or repairing an installation

If Cursor shows more than one A7S Skipper card, opens **All Dashboards** during MCP login, or
does not display the bundled rules and skill:

1. Update Cursor and install the latest A7S Skipper plugin version from the canonical EWA Team
   Marketplace entry.
2. Remove any older A7S Skipper entry that was installed from a direct Git URL or an obsolete
   team marketplace import.
3. Restart Cursor, open Settings → MCP Servers, and reconnect **a7s-skipper**.
4. Trigger an MCP tool call and complete the browser login and consent flow again.

Do not delete Cursor cache directories manually. If the duplicate remains, ask the Team
Marketplace administrator to remove the obsolete marketplace registration, then reinstall the
canonical plugin.

## OAuth troubleshooting

- **All Dashboards opens instead of returning to Cursor:** confirm that the A7S Skipper web
  deployment with the Cursor OAuth login-continuation fix is live. Then update to plugin 1.0.1 or
  newer, reconnect **a7s-skipper**, and restart OAuth from the MCP server settings.
- **Redirect URI is rejected:** update the A7S Skipper server deployment. Supported Cursor
  callbacks include the desktop custom scheme, the hosted agent callback, and the local
  `http://localhost:8787/callback` listener.
- **Consent is denied:** the account needs `catalog:read`; authoring tools additionally require
  `workspace:write`.
- **401 after a previous successful login:** reconnect the MCP server so Cursor can refresh or
  replace its stored OAuth session.

## MCP Tools

Once connected, the AI agent has access to:

**Semantic Layer** — `list_metrics`, `search_catalog`, `explore_metric` (multi-dimension `segment_by` + `filters`)

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
- **dashboard-markdown** — Dynamic text widgets with template expressions

## License

Proprietary — EWA
