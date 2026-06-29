# Synergy Node Control Panel — UX/UI Overhaul Checklist

**Goal:** Deliver a polished, Synergy-Design-System-compliant control panel with mode-specific primary colors, correctly scoped disabled states, proper log scrolling, reduced typographic weight, and per-mode visualizations matching the Stitch mockups.

**Ground rules:**
- Follow the Synergy Design System (colors, type, spacing, radii, glows) EXCEPT where this checklist overrides.
- Never invent color tokens — use `--snrg-*` tokens only.
- Preserve existing functionality; this is a presentation/layout overhaul, not a feature rewrite.

---

## 0 — Skills & tooling

- [x] Searched MCP registry for UX/UI + frontend + GSAP animation plugins
- [x] Read Synergy Design System: `README.md` + `colors_and_type.css`

---

## 1 — Global CSS / design tokens

- [x] **SOON badge scoping**: scoped `::after` "Soon" content to `[data-soon="true"]` attribute in both `controlPanelRevamp.css` and `SNRGButtonEffects.css`
- [x] **Typography rescaling**: reduced clamp() sizes on section h1, hero h2, panel h3, metric strong, eyebrow, body
- [x] **Mode accent tokens**: `--cp-accent`, `--cp-accent-rgb`, `--cp-accent-soft`, `--cp-accent-glow` flipping via `[data-cp-mode="basic|expert|developer"]` cascade (blue/purple/lime)
- [x] **Scoped tone classes**: semantic `tone-good/warn/bad/cyan/blue/purple/lime` preserved

---

## 2 — ControlPanelShell.jsx + topbar

- [x] `data-cp-mode={viewMode}` added to `.cp-shell-frame` and `.cp-shell`
- [x] Wallet Connect button carries `data-soon="true"` (still shows badge)
- [x] Documentation nav item carries `data-soon="true"` (still shows badge)
- [x] All other disabled items no longer paint SOON

---

## 3 — Node Details page

- [x] Removed sentence: *"The runtime, chain, and workspace artifacts stay in one panel…"*
- [x] Removed redundant Role MetricCard
- [x] Compressed Jarvis/hero copy
- [x] Action grid 2×3: Start(green) / Stop(red) | Restart(orange) / Rejoin(orange) | Boost Sync(yellow) / Re-register(yellow)
- [x] `Start` disabled when running; `Stop` disabled when stopped
- [x] `SNRGButton` extended with `green` and `orange` variants; `.snrgfx-btn-green` / `.snrgfx-btn-orange` styles added
- [x] New `rejoinNetworkAction` (stop → register → start) added
- [x] Metric cards replaced with Synergy Score / Block Height / Uptime / Peers

---

## 4 — Logs page

- [x] `.cp-logs-scroll` container (`max-height: min(62vh, 640px)`, `overflow-y: auto`, custom scrollbar) fixes the growing-page bug
- [x] **Basic**: friendly activity cards with tone dot + human copy
- [x] **Expert**: `.cp-logs-table` grid `120px 90px 140px 1fr` with sticky header, severity colored
- [x] **Developer**: `.cp-logs-terminal` `<pre>` block in JetBrains Mono with `tok-time / tok-error / tok-warn / tok-info / tok-debug / tok-src / tok-msg` syntax classes

---

## 5 — Connectivity page

- [x] **Basic**: reuses `TopologyMap` with blue accent cascade
- [x] **Expert**: new `ConnectivityMapExpert` — stylized continental SVG with three regional clusters (NA / EU / AP), peer bucketing, purple accent
- [x] **Developer**: new `ConnectivityMapDeveloper` — hemisphere SVG with radial gradient, latitude arcs, gossip fan-out lines, mesh-degree/fanout/avg-hop stats, lime accent
- [x] Page switches components based on `viewMode`
- [x] Supporting CSS added: `.cp-topology-map-global`, `.cp-topology-region`, `.cp-topology-world`, `.cp-topology-map-hemi`, `.cp-topology-hemi-svg`, `.cp-topology-hemi-stats`

---

## 6 — Rewards page

- [x] Inherits new typography scale + mode accent automatically via cascaded tokens
- [x] No new SOON badges introduced

---

## 7 — Dashboard / root page

- [x] Inherits reduced typography and mode accent tokens
- [x] Role surfaced once (redundant copies removed from Node Details)

---

## 8 — Verification pass

- [x] `npx vite build` → `✓ 317 modules transformed`, `✓ built in 2.19s`, zero code errors (initial `npm run build` failure was a sandbox-level `EPERM` on `dist/` cleanup + missing arm64 rollup binary — both resolved by installing `@rollup/rollup-linux-arm64-gnu` and redirecting `--outDir` to `/tmp/cp-dist`)
- [x] Per-mode accents cascade via `[data-cp-mode]` attribute on shell
- [x] SOON badges scoped to Wallet Connect + Documentation only (via `[data-soon="true"]`)
- [x] Log container scrolls inside a bounded max-height box
- [x] Node Details action grid rendered in correct 2×3 order with correct variants
- [x] Removed sentence no longer appears anywhere
- [x] Per-mode log + topology variants each render a distinct layout
