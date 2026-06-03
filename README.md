# Verslay plugin for Claude

Verslay business agents for Claude — leads, CRM, content, research, scheduling.

This plugin packages the **Verslay project instructions** together with the **Verslay
remote MCP connector** (`mcp.verslay.com`) into a single installable bundle. Once
installed, Claude becomes a power-user of Verslay: it routes a request like "find me
50 SaaS founders to email" or "why are my deals stalling?" to the right Verslay
agent, runs it through the connector, and persists what it learns to your Verslay
business memory.

The bundled skill is the **same canonical bootstrap instructions** Verslay users paste
into a claude.ai Project (downloaded from the hub as `verslay-instructions.md`) — not
Verslay's internal orchestrator master skill, which stays private to the MCP server.

## What this bundles

| Component | Path | What it does |
| --- | --- | --- |
| **Project instructions** | `skills/verslay-agent-instructions/` | `SKILL.md` — the canonical Verslay bootstrap: initialize every conversation with `verslay_initialize`, discover the right agent via `verslay_discover`, activate with `verslay_deploy`, and use `verslay_recall`/`verslay_memorize`/`verslay_report` for memory and logging. |
| **MCP connector** | `.mcp.json` | Wires the remote Verslay MCP server (`https://mcp.verslay.com/mcp`, HTTP transport) so all `verslay_*` and provider tools (HubSpot, Gmail, LinkedIn, Calendly, Slack, Stripe, and dozens more) are available the moment the plugin is enabled. |

```text
verslay-plugin/
├── .claude-plugin/
│   ├── plugin.json          ← plugin manifest (name, version, author, homepage)
│   └── marketplace.json     ← single-plugin marketplace manifest
├── .mcp.json                ← bundled remote MCP server (auto-discovered at plugin root)
├── skills/
│   └── verslay-agent-instructions/
│       └── SKILL.md         ← canonical Verslay project instructions (same as hub download)
└── README.md
```

## Install (developers / Claude Code)

The plugin is distributed as a marketplace. From a Claude Code session:

```text
/plugin marketplace add shubham-verslay/verslay-plugin
/plugin install verslay@verslay
```

- The first command registers this repo as a plugin marketplace (the
  `marketplace.json` at `.claude-plugin/marketplace.json` defines the `verslay`
  marketplace and its single `verslay` plugin).
- The second installs the `verslay` plugin from the `verslay` marketplace, enabling
  the master skill and starting the bundled MCP connector.

You can also point at a local checkout instead of a remote source while iterating:

```text
/plugin marketplace add /absolute/path/to/verslay-plugin
/plugin install verslay@verslay
```

After enabling or disabling the plugin mid-session, run `/reload-plugins` so its MCP
server connects or disconnects.

### Authorizing the connector

The bundled MCP server is the Verslay remote connector. It authenticates per-user
via OAuth — on first use Claude will prompt you to authorize it with your Verslay
account (run `/mcp` in Claude Code to complete the OAuth flow). No API key is baked
into this bundle. You need a Verslay account; manage connected tools and agents at
`hub.verslay.com`.

## Submitting to the official plugin directory

This bundle is built for submission to Anthropic's official plugin directory, which
syncs approved plugins into the **Plugins** tab on claude.ai. Submit it through:

> **clau.de/plugin-directory-submission**

Submission reviews the plugin manifest, the bundled skill, and the MCP connector.
Keep `version` in `plugin.json` bumped on every change you want users to receive —
pushing new commits without bumping the version has no effect once a version is set.

## Notes

- The plugin name is strict kebab-case (`verslay`) and is used for namespacing.
- The MCP server is wired with `"type": "http"` (Claude also accepts the
  `streamable-http` alias) pointing at `https://mcp.verslay.com/mcp`.
- Plugin components (the `skills/` directory and `.mcp.json`) live at the plugin
  root, not inside `.claude-plugin/` — only the manifests live in `.claude-plugin/`.
