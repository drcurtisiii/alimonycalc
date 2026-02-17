# Chart Style (TechPulse Stock Analytics)

## Goal
Recreate the **overall visual style** of this app for another app: a polished fintech dashboard with soft card surfaces, subtle grid background, gradient accents, and clean chart cards. The app supports light and dark themes via CSS variables and applies consistent typography, spacing, and hover/animation behavior across UI and charts.

## Typography & Fonts
- Primary UI font: `DM Sans` (Google Fonts). Used for body, headings, labels.
- Monospace accent font: `JetBrains Mono` (Google Fonts). Used for tickers, numeric data, buttons, and chart labels.
- Typical sizing: 0.75rem–1.5rem labels, 2rem for primary metrics (ticker + price).

## Color System (CSS Variables)
All color styling is defined as CSS custom properties and swapped by toggling `.theme-dark` on `body` or a top-level container.

Light theme (`:root`):
- Backgrounds: `--bg-primary #f4f6fb`, `--bg-secondary #ffffff`, `--bg-card #ffffff`, `--bg-card-hover #eef1f7`
- Borders: `--border-color #d7dbe6`
- Text: `--text-primary #111427`, `--text-secondary #3f435c`, `--text-muted #6e738c`
- Accents: `--accent-green #00b894`, `--accent-red #e64c5c`, `--accent-blue #2d7dff`, `--accent-purple #7c3aed`
- Accent dims: `--accent-green-dim rgba(0,184,148,0.15)`, `--accent-red-dim rgba(230,76,92,0.15)`
- Gradients: `--gradient-start #00b894`, `--gradient-end #2d7dff`
- Chart colors: `--grid-line rgba(24,24,40,0.1)`, `--chart-tick #3c4161`, `--chart-grid rgba(26,26,46,0.12)`, `--chart-tooltip-bg #ffffff`, `--chart-tooltip-border #d7dbe6`, `--chart-tooltip-body #2b2f45`, `--chart-tooltip-title #0f1224`, `--chart-point-border #ffffff`

Dark theme (`.theme-dark`):
- Backgrounds: `--bg-primary #0a0a0f`, `--bg-secondary #12121a`, `--bg-card #16161f`, `--bg-card-hover #1a1a25`
- Borders: `--border-color #34344a`
- Text: `--text-primary #f0f0f5`, `--text-secondary #b6b6cc`, `--text-muted #8a8aa3`
- Accents: `--accent-green #00d4aa`, `--accent-red #ff4757`, `--accent-blue #4a9eff`, `--accent-purple #a855f7`
- Accent dims: `--accent-green-dim rgba(0,212,170,0.15)`, `--accent-red-dim rgba(255,71,87,0.15)`
- Gradients: `--gradient-start #00d4aa`, `--gradient-end #4a9eff`
- Chart colors: `--grid-line rgba(42,42,58,0.3)`, `--chart-tick #9a9ab5`, `--chart-grid rgba(90,90,120,0.45)`, `--chart-tooltip-bg #1a1a25`, `--chart-tooltip-border #34344a`, `--chart-tooltip-body #d0d0e6`, `--chart-tooltip-title #f0f0f5`, `--chart-point-border #16161f`

## Background Treatment
- Fixed, subtle grid overlay on the `body`:
  - Two layered linear gradients (horizontal + vertical lines)
  - `background-size: 60px 60px`
  - `pointer-events: none` and `z-index: -1`
- This creates a clean, technical backdrop without distracting from cards.

## Layout + Surfaces
- Container: max-width 1400px, horizontal padding ~2rem, top padding ~1.25rem.
- Sections are card-based with rounded corners and thin borders.
- All cards use: `background: var(--bg-card); border: 1px solid var(--border-color); border-radius: 16px`.
- Hover states emphasize depth: card background changes to `--bg-card-hover`, border shifts to `--accent-blue`, and shadow adds subtle elevation.

## Header + Controls Styling
- App header uses a left-aligned logo block with gradient icon and wordmark.
- The logo icon is a rounded square (10px radius) with a 135° gradient background.
- Primary controls (theme toggle, stock select) are pill-like buttons:
  - `border-radius: 12px`, `padding: 0.75rem–0.875rem 1–1.25rem`
  - monospace font, medium weight, subtle hover background
  - hover: border to accent blue, background to hover color

## Info Banner (Stock Summary)
- `stock-info` is a card-like banner with inline blocks: ticker, price, change, and small stats.
- Price and ticker are 2rem, monospace, semi-bold.
- Change pill uses green/red text with matching translucent backgrounds.

## Chart Grid + Cards
- Grid layout: 3 columns; breaks to 2 @ 1100px, 1 @ 700px.
- Each `chart-card`:
  - Rounded 16px, padding ~1.1–1.25rem
  - `cursor: pointer`
  - Staggered `fadeIn` animation on load (0.1–0.35s delay)
  - Hover: lift `translateY(-4px)` and drop shadow `0 12px 40px rgba(0,0,0,0.4)`
- Chart header includes:
  - Title with tiny gradient dot (`8px` circle)
  - Subtle “click to expand” hint (hidden until hover)
  - Change pill with green/red accent background
- Chart area height is 160px for mini charts.

## Modal Expansion Style
- Fullscreen overlay with dark translucent background + blur.
- Modal panel is a large card with rounded 20px corners, border, and scale-in transition.
- Close button: rounded square, border, red hover treatment.
- Expanded chart container height: 400px.

## Animations
- `fadeIn`: cards animate upward with opacity.
- `slideIn`: stock info banner enters from above.
- `spin`: loading spinner for charts.
- Transitions are short and smooth (0.2–0.5s).

## Chart.js Styling Details
Chart colors are driven by CSS variables pulled at runtime:
- `tickColor`: `--chart-tick`
- `gridColor`: `--chart-grid`
- Tooltip: `--chart-tooltip-bg`, `--chart-tooltip-border`, `--chart-tooltip-body`, `--chart-tooltip-title`
- `pointBorder`: `--chart-point-border`

Mini chart (card) styles:
- Line chart, smooth tension `0.4`, gradient fill.
- Line color: green if period is up (`#00d4aa` in dark, `#00b894` in light) or red if down (`#ff4757` / `#e64c5c`).
- Fill gradient from ~0.3 alpha to transparent.
- Points hidden (`pointRadius: 0`) but larger hover radius.
- Y-axis ticks on right side, small monospace font.
- X-axis hidden.

Modal chart styles:
- Similar line chart with thicker stroke and visible points if dataset small.
- Tooltip uses monospace font and shows formatted date + price.
- X-axis ticks display date labels and vary by time range (days/months/years).

## Implementation Notes
- Theme switch is applied by toggling `.theme-dark` on a top-level element; no other CSS changes needed.
- All component styling is driven by the variable set, so new apps should adopt the same variable names to mirror the look.
- For hover/active feedback, use `transform: scale(0.98)` on controls and mild lift on cards.

## File Reference (source)
- Style and behavior come from `index.html` in this project.
