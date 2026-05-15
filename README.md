# Retention & Expansion Intelligence

A live, single-file HTML dashboard that surfaces customer success process effectiveness — built as a [Cowork](https://claude.com/) artifact that pulls live data from Gainsight via MCP and falls back to a pre-analyzed snapshot when the connector is unavailable.

The dashboard turns raw CTA (Call-to-Action) data into operational intelligence: which Journey Orchestrator programs are working, which are quietly failing, which proven plays have evidence behind them, and which new plays the data suggests you should build.

---

## Preview

The dashboard ships as a single `index.html` file with four tabs:

| Tab | What it shows |
| --- | --- |
| **CTA Effectiveness** | Close-rate matrix across every CTA Type × Reason combination, plus a process-gap analysis |
| **Journey Orchestrator** | Program-by-program breakdown of what's firing reliably vs. backlog-only |
| **Proven Plays** | Six retention/expansion motions backed by Gainsight + Slack evidence |
| **Recommended Plays** | AI-generated plays with one-click "Create as Journey Program" / "Create Playbook" actions |

---

## Features

- **Live data via MCP** — calls `mcp__...__run_query` against the Gainsight `call_to_action` object on load and on refresh, with automatic fallback to a static snapshot if the connector is unreachable
- **Zero build step** — pure HTML/CSS/JS in one file, ready to serve from any static host or render inside Cowork
- **Modern, accessible UI** — Inter typography, Lucide-style inline SVG icons, semantic color tokens, tabular-nums for clean number alignment, responsive grid breakpoints
- **One-click asset creation** — every recommended play has a button that sends a fully-specified prompt back to Claude via `window.cowork.sendPrompt()` to walk you through creating the Journey Orchestrator program or Playbook in Gainsight
- **Self-contained** — no external CSS frameworks, no bundlers, no npm dependencies. Only network call out of the page is Google Fonts for Inter + JetBrains Mono

---

## Tech stack

- HTML + vanilla CSS (CSS custom properties for theming) + vanilla JS
- [Inter](https://rsms.me/inter/) and [JetBrains Mono](https://www.jetbrains.com/lp/mono/) via Google Fonts
- [Cowork artifact runtime](https://claude.com/) for `window.cowork.callMcpTool` and `window.cowork.sendPrompt`
- Gainsight MCP server for the live `run_query` call against the `call_to_action` object

---

## File structure

```
.
├── index.html        # The entire dashboard — markup, styles, scripts, static data
└── README.md         # This file
```

The `<script type="application/json" id="cowork-artifact-meta">` block at the top of `index.html` declares the artifact name, MCP tools, and connector requirements that Cowork uses to wire the artifact into a workspace.

---

## Getting started

### Run it as a Cowork artifact

1. Open Cowork and upload `index.html`, or call `create_artifact` with the file contents.
2. Make sure the Gainsight MCP server (`Gainsight-CSMCP-Demo` or your equivalent) is connected.
3. Open the artifact — it will auto-load CTA data on first render and stamp the header with the refresh time.

### Run it as a static page

The dashboard works as a regular static page too — useful for screenshots, demos, or hosting on GitHub Pages. The MCP calls will fail gracefully and the page will render against the embedded `STATIC_CTA_DATA` snapshot.

```bash
# Any static server works
python3 -m http.server 8000
# then visit http://localhost:8000
```

---

## Configuration

### Pointing at a different Gainsight tenant

Update the `mcpTools` and `mcpServerNames` entries in the metadata block at the top of `index.html`:

```html
<script type="application/json" id="cowork-artifact-meta">
{
  "name": "Retention Expansion Intelligence",
  "schemaVersion": 1,
  "mcpTools": ["mcp__<your-server-id>__run_query"],
  "mcpServerNames": ["<Your-Gainsight-MCP-Name>"]
}
</script>
```

Then update the matching `window.cowork.callMcpTool(...)` call inside `loadAllData()` so the tool name lines up.

### Adjusting the query

The CTA query lives in `loadAllData()`. It groups by `TypeId__gr.Name`, `ReasonId__gr.Name`, and `StatusId__gr.Name` and counts CTAs. Edit the `select`, `group_by`, or `sort_by` arrays to reshape the dataset — `renderEffectiveness()` normalizes field names, so as long as the rows expose `CTAType`, `Reason`, `Status`, and `Count`, the rest of the dashboard keeps working.

### Refreshing the static fallback

`STATIC_CTA_DATA` is the snapshot the dashboard uses when MCP is unavailable. Replace it with the output of a fresh `run_query` whenever the underlying data drifts significantly — the schema is an array of `{CTAType, Reason, Status, Count}` objects.

### Editing the plays

Proven plays live in the `#tab-proven` section and recommended plays in `#tab-recommended`. Each card is plain HTML — change copy, evidence quotes, metric numbers, and step lists in place. The action buttons accept a free-form description string that gets sent back to Claude via `sendPrompt()`, so be specific about triggers, conditions, and expected outcomes.

---

## How the action buttons work

Every "Create Journey Program" or "Create Playbook" button calls one of two helpers:

```js
function createPlay(description) {
  window.cowork.sendPrompt('Please create a Journey Orchestrator program in Gainsight based on this description: ' + description);
}

function createPlaybook(description) {
  window.cowork.sendPrompt('Please help me set up a Gainsight playbook with the following design: ' + description + '. Walk me through the CTA setup, task structure, and any automation rules needed.');
}
```

`window.cowork.sendPrompt` sends a message into the Cowork chat as if the user typed it, kicking off an agent conversation to actually create the asset. Outside of Cowork these calls are no-ops, so the dashboard remains safe to demo as a static page.

---

## Customization tips

- **Theme colors** — every color is a CSS variable at the top of the `<style>` block. Swap `--accent`, `--success`, `--danger`, etc. to match your brand
- **Tabs** — to add a tab, add a `.tab` div in `.tabs`, a matching `.tab-content` block, and extend the `ids` array inside `showTab()`
- **Charts** — the original version of this artifact pulled Chart.js from a CDN. The redesigned version uses pure CSS for bars; reintroduce Chart.js by adding the `<script src=...>` back if you want trended visualizations

---

## Data sources

- **Gainsight** — Cockpit CTA data (Type, Reason, Status), Company health, ARR, renewal stages
- **Staircase AI** — engagement signals (referenced in the "Dark Account Reactivation" recommended play)
- **Slack** — deal-win evidence cited in the Proven Plays (#sfdc-ring-the-gong, #teammate_announcements)

Only Gainsight is queried live by the dashboard today; the Staircase and Slack data points are baked into the proven/recommended play copy as evidence.

---

## Browser support

Modern evergreen browsers (Chrome, Edge, Firefox, Safari). Uses CSS custom properties, CSS grid, `backdrop-filter`, and `font-feature-settings` — no fallback layer is shipped.

---

## License

Internal Gainsight asset — adapt freely within your organization.
