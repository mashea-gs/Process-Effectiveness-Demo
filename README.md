# Process Effectiveness

A lightweight HTML dashboard that puts proven and AI-recommended customer success plays in one place — and lets you spin up the matching Gainsight asset with a single click.

Built as a [Cowork](https://claude.com/) artifact. Each play card is a self-contained brief; each "Create" button hands the play description to Claude, which then walks you through building the Journey Orchestrator program or Playbook in Gainsight.

## Contents

- [Why this exists](#why-this-exists)
- [What's inside](#whats-inside)
- [Running it](#running-it)
- [Editing the plays](#editing-the-plays)
- [Theming](#theming)
- [How the buttons work](#how-the-buttons-work)
- [Project layout](#project-layout)

## Why this exists

Customer success teams collect mountains of evidence — closed-won deals in Slack, Cockpit CTA outcomes, EBR signals — but rarely turn that evidence into reusable plays. This artifact does two things at once:

1. **Documents the plays your team has already proven** — with the deal, the quote, the metric.
2. **Surfaces AI-suggested plays your data implies you should be running** — with trigger conditions and expected outcomes spelled out.

Every play has a button that turns the brief into an action: a chat with Claude that builds the Journey Orchestrator program or Playbook for you, step by step.

## What's inside

Two tabs, both rendered from static markup baked into the page:

**Proven Plays.** Six retention/expansion motions with verified outcomes — Bundled Renewal + Expansion (Tanium $615K), Overage CTA Automation (Mailgun 14 deals/$88K), Staircase AI Cross-Sell ($332K by a single CSM), Product Risk Resolution (178 closed CTAs), EBR-Triggered Expansion Discovery (100% close rate), and Customer Communities SFDC Displacement.

**Recommended Plays.** Six AI-generated suggestions with trigger conditions and expected outcomes — Renewal Velocity Program, Upsell Pipeline Conversion Sprint, Dark Account Reactivation, CSQL Automation, Product Risk → Expansion Bridge, and Multi-Product Renewal Bundling.

Each card includes a "Create Journey Program" button (primary) and a "Create Playbook" button (secondary).

## Running it

**In Cowork.** Upload `index.html` or pass it to `create_artifact`. Open the artifact. Click a button. That's the whole loop.

**As a static page.** It works anywhere — file system, GitHub Pages, S3, Netlify, `python3 -m http.server`. The Cowork-specific button handlers become no-ops outside the Cowork runtime, so nothing breaks; the buttons just won't open a chat.

There is no build step, no `npm install`, no bundler. The only network request the page makes is to Google Fonts for Inter and JetBrains Mono.

## Editing the plays

All play content lives directly in `index.html` — no JSON, no CMS, no templating. Search for the play title, edit the surrounding HTML.

**To change a proven play's metrics**, edit the three `<div class="play-metric">` blocks inside its `metrics-bar`.

**To change a recommended play's button prompt**, edit the string passed to `createPlay(...)` or `createPlaybook(...)`. That string is the exact prompt Claude receives, so be specific about triggers, thresholds, escalation rules, and outcomes.

**To add a new play**, copy an existing `.play-card` or `.rec-play-card` block and paste it inside the appropriate grid. The card width auto-balances; the icon colors are limited to the predefined classes (`blue`, `green`, `purple`, `orange`, `teal`, `pink`).

## Theming

Every color, radius, and shadow is a CSS custom property declared in `:root` at the top of the `<style>` block. To rebrand:

```css
:root {
  --accent: #4f46e5;       /* primary actions, links, focus states */
  --accent-soft: #eef2ff;  /* tinted backgrounds */
  --success: #079455;
  --warning: #b54708;
  --danger:  #b42318;
}
```

Typography is set on `body` — swap the `font-family` declaration to change the entire dashboard.

## How the buttons work

Both action helpers are thin wrappers around `window.cowork.sendPrompt`:

```js
function createPlay(description) {
  window.cowork.sendPrompt(
    'Please create a Journey Orchestrator program in Gainsight based on this description: ' + description
  );
}

function createPlaybook(description) {
  window.cowork.sendPrompt(
    'Please help me set up a Gainsight playbook with the following design: ' + description +
    '. Walk me through the CTA setup, task structure, and any automation rules needed.'
  );
}
```

`sendPrompt` injects a message into the active Cowork chat as if the user had typed it. From there, Claude handles asset creation conversationally — asking for missing details, confirming trigger logic, and posting back when the asset is built.

## Project layout

```
.
├── index.html     # The entire dashboard
└── README.md      # This file
```

That's it. One file to ship, one file to document, one place to edit.

## Browser support

Modern evergreen browsers — Chrome, Edge, Firefox, Safari. Uses CSS custom properties, CSS grid, `backdrop-filter`, and `font-feature-settings`. No IE fallback.

## License

Internal Gainsight asset. Adapt freely within your organization.
