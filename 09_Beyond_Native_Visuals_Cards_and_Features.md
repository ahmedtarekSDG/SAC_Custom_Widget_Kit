# 09 — Beyond Native SAC: Visuals, Card Types and Features Worth a Custom Widget

This chapter answers one question: **which visuals, KPI cards and interactions are missing (or weak) in native SAC,
where does each one earn its place, and how should it be built and compared?** It complements the
[Possibilities Catalog](05_Possibilities_Catalog.md) (what is technically possible) with design judgement: what to
build first, for which business area, and which comparison visual to pick for which question.

It assumes stories on **SAP Datasphere analytic models** (live), so every proposal states the data shape the
widget needs from the model.

Legend used throughout:

| Symbol | Meaning |
|---|---|
| ● | Native in SAC (optimized story experience, status QRC3 2026) |
| ◐ | Partly native: a workaround exists (combination of widgets, reference lines, scripting) but with gaps |
| ○ | Not native |
| ✅ | Already in this kit |
| ⚙ | Possible with a kit widget plus configuration (`optionOverride`, extra measures) |
| 🛠 Bn | Recommended new build, see backlog item Bn in [section 9](#9-build-backlog-for-the-kit) |
| — | Not worth building (use native or skip) |

> The native catalogue changes every quarter. Re-check section 2 against "What's New" for your tenant's release
> before committing to a build (checklist in [section 11](#11-verify-before-you-build)).

---

## 1. How to use this chapter

1. Start from the **business question** (section 4 groups visuals by the question they answer, following the
   Financial Times Visual Vocabulary categories).
2. Check the **native status**. If it is ●, use native and stop; the custom widget would lose theme, commenting,
   scheduling, translation and other story features (restrictions in
   [01 — Master Guide](01_Custom_Widget_Master_Guide.md)).
3. If it is ◐ or ○, read the area playbook (section 8) and the comparison guide (section 7) to confirm the
   visual is the right one, then take the backlog item (section 9) or a ready recipe (section 10).

---

## 2. The native baseline in 2026 (do not rebuild these)

| Area | Native in SAC | Notes for custom-widget decisions |
|---|---|---|
| Comparison | Bar/column, combination column + line, combination stacked column + line, stacked bar/column, waterfall | Waterfall gaps (PVM across measures, hierarchy bridges, subtotal control) are covered by the kit's bridge, see [04](04_Waterfall_and_Bridge_Guide.md) |
| Trend | Line, stacked area, time series (with forecast and confidence band) | Time series forecast is statistical; scenario bands from plan versions are not native |
| Distribution | Box plot, heat map, histogram, radar, tree map | No violin, beeswarm, calendar heatmap |
| Correlation | Scatterplot, bubble, cluster bubble | No quadrant shading or named zones; reference lines only |
| Indicator | Numeric point, bullet | Numeric point shows primary/secondary values and variances with threshold colours |
| Part-to-whole | Pie, donut, marimekko, tree map, stacked 100 % | No sunburst, waffle |
| Flow and new types (QRC3 2026) | **Sankey, gauge, funnel** (added with the move of the chart engine to Highcharts 12) | Older stories and tenants on earlier releases do not have them; the kit's Universal Chart keeps Sankey/funnel/gauge for those and adds link-level linked analysis |
| Layout | Trellis (small multiples), responsive lanes, panels, popups | |
| Tables | Hierarchies, thresholds, in-cell charts (bar, variance bar, variance pin), comments, data entry | No line sparkline or bullet column per row |
| Variance and IBCS | Variance in bar/column charts (incl. integrated variances), variance waterfall, IBCS templates and scenario patterns (solid actual, outlined plan, hatched forecast); SAC is IBCS-certified | Gaps are in the finer IBCS composition rules (shared scales across charts, multi-tier charts), see section 4.10 |
| Reuse | **Composites**: reusable widget groups with interface functions and events, transportable, data source re-mapping (QRC2 2024), copy/paste across stories (QRC3 2026) | A "card" built from numeric point + chart inside a composite is the first thing to try before a custom card |
| KPI lists | **My Metrics** (QRC4 2025): personal KPI lists with standard visualisations, shareable | Personal monitoring, not a story layout element |
| Geo | Geo map layers (bubble, choropleth, heat, flow, point of interest) | Custom shapes (plant floor, warehouse) are not native |
| Planning | Value Driver Tree (on SAC planning models) | Not an ad-hoc decomposition tree and not available on a Datasphere analytic model |

Also coming from SAP itself: the **Custom Widget Library** (Business Data Cloud tenants only) lists demand areas such
as hierarchy analysis, project timelines, diverging data, attainment and multi-KPI views, with dumbbell, dotted
lines, trend/forecast overlay, org chart and tornado announced as upcoming. If you are on BDC, check the library
before building any of those (see [08 — Resource Library](08_Resource_Library.md)).

---

## 3. When a custom widget is justified

Climb the ladder and stop at the first step that answers the question:

| Step | Use when | Cost |
|---|---|---|
| 1. Native widget | The visual exists natively (section 2) | None |
| 2. Composite of native widgets | You need a repeatable card or header (numeric point + small chart + text) | Each inner widget sends its own query; many cards = many queries |
| 3. Widget add-on | Native chart is right but needs a custom tooltip or a drawn overlay on bar/column/line/stacked/area or numeric point | Limited chart types; no new data |
| 4. Custom widget (kit-based) | The encoding, layout density or interaction does not exist natively | Story features lost (theme, commenting, scheduling, translation, Search to Insight …); you own the code |
| 5. Vendor extension | IBCS suites, Gantt or map products with support contracts | Licence, hosting, vendor release cycle |

A custom widget is justified when **at least one** of these is true:

- **Encoding gap**: the question needs a mark SAC does not draw (dumbbell gap, slope crossing, Gantt bar, chord
  ribbon, quadrant zone).
- **Density gap**: one widget must show what would otherwise take 3–6 native widgets and queries (multi-KPI card,
  micro-chart table). This is also a performance argument: one bound query instead of six.
- **Interaction gap**: brushing a range, expanding a tree node by node, client-side what-if, cross-highlighting
  without filtering.
- **Standardisation**: the same non-native visual is needed in many stories (finance pack, group reporting pack),
  so a governed widget beats copy-pasted workarounds.

It is **not** justified for decoration (3D, liquid fill, glossy gauges), for a visual the audience will not read
without training, or when the native version differs only in styling (use theme/CSS first, see the sac-css notes
in [01](01_Custom_Widget_Master_Guide.md)).

---

## 4. Gap matrix by business question

Organised by the nine Financial Times Visual Vocabulary relationships (deviation, correlation, ranking,
distribution, change over time, magnitude, part-to-whole, spatial, flow) plus business-specific forms.
"Priority" is for a finance-led SAC programme on Datasphere: P1 = build first, P2 = build on demand, P3 = only
for a specific use case.

### 4.1 Deviation (emphasise +/- from a reference)

| Visual | Answers | Native | Build | Priority | Typical areas |
|---|---|---|---|---|---|
| Variance bar (diverging bars, AC − PL) | Which items are above/below plan and by how much | ● | — | — | All |
| Integrated variance (AC bars with ΔPL overlay) | Values and variance in one chart | ● | — | — | Finance |
| Variance pin / lollipop for Δ % | Relative variance, many items | ● (in-cell), ◐ (chart) | 🛠 B1 | P1 | Finance, sales |
| **Multi-tier variance chart** (AC/PL bars + ΔPL bars + ΔPL % pins on one shared category axis) | Absolute and relative variance in one IBCS-style view | ◐ (three native charts, scales not synchronised) | 🛠 B1 | P1 | Finance, group reporting |
| Surplus/deficit area (area between actual and target coloured by sign) | When did we run above/below target over time | ○ | ⚙ / 🛠 B1 | P2 | Cash vs plan, budget burn, capacity |
| Butterfly / tornado | Two opposing groups per category, or sensitivity ranking | ○ | 🛠 B12 | P2 | HR pyramid, sensitivity analysis, import/export |
| Diverging stacked bar (Likert) | Survey results around a neutral midpoint | ◐ (stacked with negatives) | 🛠 B12 | P3 | HR engagement, customer surveys |

### 4.2 Correlation (relationship between two or more measures)

| Visual | Answers | Native | Build | Priority | Typical areas |
|---|---|---|---|---|---|
| Scatter / bubble | Relationship and outliers | ● | — | — | All |
| **Quadrant / 2×2 matrix** (scatter with shaded, named zones, medians or targets as dividers, size = spend) | Which segment does each item fall into and what is the action | ◐ (scatter + reference lines, no zone shading or labels) | ⚙ (`markArea`) / 🛠 B3 | P1 | Procurement Kraljic, BCG portfolio, customer value vs growth, risk matrix |
| Connected scatter | How two measures moved together over time | ○ | ⚙ | P3 | Price vs volume path, margin vs utilisation |
| Combination (columns + line, two axes) | Two measures over time | ● | — | — | All |

### 4.3 Ranking (position matters more than value)

| Visual | Answers | Native | Build | Priority | Typical areas |
|---|---|---|---|---|---|
| Ordered bar, Top N | Who is biggest | ● | — | — | All |
| Table with in-cell bars | Ranked list with exact values | ● | — | — | All |
| **Dumbbell** (two dots per category joined by a bar) | How big is the gap between two values per item (AC vs PL, PY vs AC, min vs max) | ○ (announced for the BDC library) | 🛠 B2 | P1 | Cost centres AC vs budget, regions PY vs AC, price ranges |
| **Slope chart** | Direction of change and rank swaps between two points | ◐ (line chart over two periods, no rank labels on both ends) | 🛠 B8 | P2 | Market share, entity margins, supplier scores |
| Bump chart | Rank changes over several periods | ◐ (line on a rank measure with inverted axis via script) | 🛠 B8 | P3 | League tables, product ranking |
| Lollipop / dot plot | Ranking with less ink than bars, many items | ○ | ⚙ | P3 | Long ranked lists, mobile |

### 4.4 Distribution (spread and shape)

| Visual | Answers | Native | Build | Priority | Typical areas |
|---|---|---|---|---|---|
| Histogram, box plot | Spread, median, outliers | ● | — | — | All |
| Strip / beeswarm (box plot with every point) | Where individual items sit within the spread | ○ | ⚙ (scatter on a category axis) | P3 | Salary bands, DSO per customer, lead times |
| Population pyramid | Two distributions back to back | ○ | 🛠 B12 | P2 | HR age/gender, tenure |
| **Calendar heatmap** | Day-of-week and seasonal patterns over a year | ○ | 🛠 B11 | P2 | Daily sales, cash receipts, downtime, absence |
| Cumulative curve / Lorenz | Concentration (what % of items make X % of value) | ◐ (running sum line) | ⚙ (Pareto) ✅ | — | Spend concentration, customer concentration |

### 4.5 Change over time

| Visual | Answers | Native | Build | Priority | Typical areas |
|---|---|---|---|---|---|
| Line, column, area, time series forecast | Trend | ● | — | — | All |
| **Fan / range band** (base line with best/worst scenario band from plan versions) | How wide is the outlook range | ◐ (statistical forecast band only) | ⚙ (stacked areas) / 🛠 B1 option | P2 | Rolling forecast, cash forecast, demand plan |
| **Gantt / timeline** | Plan vs actual dates, overlaps, critical path, today line | ○ | 🛠 B4 | P1 | Projects, capex, financial close calendar, campaigns, maintenance |
| **Control chart (SPC)** with mean, UCL/LCL and rule violations marked | Is the process stable or did something change | ◐ (line + reference lines; no per-point rule highlighting) | ⚙ (UCL/LCL as measures) / 🛠 B9 | P2 | Quality, lead time, close duration, invoice cycle time |
| Burn-up / run-rate vs target | Will we reach the year-end target at the current pace | ◐ (running sum + target line) | 🛠 B6 card mode | P2 | Sales quota, budget consumption, savings tracking |
| Sparkline column in a table | Trend per row in a dense list | ◐ (in-cell bar/variance only) | 🛠 B7 | P1 | Account lists, product lists, entity lists |

### 4.6 Magnitude (size comparisons)

| Visual | Answers | Native | Build | Priority | Typical areas |
|---|---|---|---|---|---|
| Bar, grouped bar | How big, side by side | ● | — | — | All |
| Bullet graph | Value vs target within qualitative ranges | ● (single bullet) | 🛠 B6 (bullet list mode) | P2 | KPI lists, working capital days vs target |
| Marimekko | Size in two dimensions | ● | — | — | Market maps |
| Radial bar / progress ring | Attainment in a small round tile | ○ | ⚙ (polar bar) | P3 | Mobile cards only |
| Waffle (100-cell grid) | % attainment that non-experts read instantly | ○ | 🛠 B6 option | P3 | Mobile, executive tiles |

### 4.7 Part-to-whole

| Visual | Answers | Native | Build | Priority | Typical areas |
|---|---|---|---|---|---|
| Stacked / 100 % bars, pie, donut, tree map | Composition | ● | — | — | All |
| Sunburst | Composition across hierarchy levels | ○ | ✅ Universal Chart | — | Cost structure, product hierarchy |
| Bridge / waterfall incl. PVM | How a start value becomes an end value | ● (basic) | ✅ Waterfall/Bridge | — | Finance, sales |
| Composition change (two 100 % bars or a mini Marimekko per period) | Did the mix shift | ◐ | — | — | Product mix, channel mix |

### 4.8 Spatial

| Visual | Answers | Native | Build | Priority | Typical areas |
|---|---|---|---|---|---|
| Choropleth, bubble, heat, flow layers | Where | ● | — | — | Sales, logistics |
| **Floor plan / custom SVG shapes** (plant, warehouse, store, building) with bound status | Which line/bay/zone has a problem | ○ | 🛠 B14 | P2 | Manufacturing, warehousing, facilities |
| Tile grid map (equal-size country/region tiles) | Fair comparison of small and large regions | ○ | ⚙ (heatmap on row/column attributes) | P3 | GCC/MEA country views |

### 4.9 Flow

| Visual | Answers | Native | Build | Priority | Typical areas |
|---|---|---|---|---|---|
| Sankey | Volumes between states or stages | ● (QRC3 2026) | ✅ (older tenants, link-level LA) | — | P&L flow, headcount movements, customer tiers |
| Funnel | Stage drop-off | ● (QRC3 2026) | ✅ | — | Sales pipeline, recruiting |
| **Chord** | Flows between members of the same set (entity ↔ entity) | ○ | 🛠 B10 | P2 | Intercompany flows, inter-plant transfers |
| Network graph | Relationships and clusters | ○ | ⚙ (ECharts graph) | P3 | Supplier networks, shared customers |
| Pipeline waterfall (opening pipeline → new → won → lost → slipped → closing) | Why pipeline changed | ● (waterfall) | ✅ bridge with measures | — | Sales |

### 4.10 Business-specific forms

| Visual | Answers | Native | Build | Priority | Typical areas |
|---|---|---|---|---|---|
| IBCS scenario notation (solid AC, grey PY, outlined PL, hatched FC) | Consistent semantics | ● (templates, patterns) | — | — | Finance |
| **Shared scale across charts** (same value = same length on every chart of a page) | Honest visual comparison between small multiples or entity pages | ◐ (fixed axis ranges by hand) | 🛠 B1 (`scaleGroup` property + script) | P1 | Entity pages, segment pages |
| **Decomposition / driver tree** (ad hoc: split a KPI by any dimension node by node; or formula tree: ROCE = margin × turnover) | Where does a variance come from | ◐ (Value Driver Tree on planning models only) | 🛠 B5 | P1 | Finance variance root cause, OTIF tree, margin tree |
| Org chart / entity structure (with ownership %, consolidation method) | Who reports to whom, group structure | ○ (announced for BDC library) | ⚙ (tree) / 🛠 B5 layout | P2 | HR, group reporting |
| Status / exception board | Which items need action now | ◐ (table + thresholds) | 🛠 B13 | P2 | Close tasks, IC reconciliation, supplier alerts |
| Formatted financial statement grid (IBCS table: scenario headers, Δ columns with in-cell bars, bold sums, export to formatted Excel) | Statement-style reporting with variances | ◐ | ✅ Export Table + 🛠 B7 | P1 | P&L, balance sheet, cash flow |

---

## 5. Card types (KPI tiles)

### 5.1 Anatomy of a good card

```
┌────────────────────────────────────────────┐
│ Net revenue · mEUR · 2026-YTD Oct          │  title: what, unit, when (IBCS "what/when")
│ 42.7                    ▲ +3.1  +7.8 % ΔPY │  headline value · variance abs and % with sign
│ ███████████████████░░░  92 % of PL         │  context: progress / bullet vs target
│ ╱╲_╱‾╲__╱‾‾╲╱‾  (12 months, PY dashed)     │  context: sparkline with comparison
│ ● Amber · below plan by 3.6                │  status: colour + word/icon, not colour only
└────────────────────────────────────────────┘
```

Rules that apply to every card type:

1. **Never a bare number.** Every value needs a reference: target, prior period or average. A number without a
   comparison invites the reader to guess.
2. **Make the comparison window explicit** ("vs PY YTD", "vs PL Oct") and keep it the same across a card row.
3. **Variance with sign and favourability.** Show `+`/`−`, colour by *favourability*, not by sign (an increase in
   cost is bad): the kit's `increaseIsGood` flag.
4. **Status = colour + shape or word.** Colour-blind users and grey-scale prints must still read it.
5. **Units and scale in the title**, not on every number.
6. **Five to seven cards per row at most**; fewer, better-chosen KPIs beat a wall of tiles.
7. **Same layout for every card in a row** (value position, variance position) so the eye scans one line.
8. **Click leads somewhere**: filter the page (linked analysis) or open the detail (popup/page).
9. **Prefer bullets over gauges.** The bullet graph was designed to replace dials: linear, compact, with one or two
   comparative markers and two to five qualitative ranges; keep ranges in shades of one hue and the actual bar
   narrower than the bands.
10. **Accessible text**: an `aria-label` that reads like a sentence ("Revenue 42.7 m EUR, 92 % of plan, amber").

### 5.2 Card catalogue

| # | Card type | Shows | Native | Kit | Best for |
|---|---|---|---|---|---|
| C1 | Big number | Value only | ● numeric point | ✅ | Only with a comparison next to it; otherwise avoid |
| C2 | Comparison card | Value, Δ abs/%, status colour | ● numeric point (variance + thresholds) | ✅ KPI Card | Any headline KPI |
| C3 | **Trend card** | C2 + sparkline with PY/PL line | ◐ composite (numeric point + line chart, two queries) | ✅ KPI Card | Revenue, margin, cash, headcount |
| C4 | **Target / progress card** | Value vs target with progress bar or bullet, % attainment | ◐ composite (numeric point + bullet) | ✅ (progress bar) / 🛠 B6 (bullet mode) | Quota, budget consumption, savings |
| C5 | **Landing / forecast card** | YTD actual + rest-of-year forecast = full-year landing vs budget, as one stacked bar with a budget tick | ○ | 🛠 B6 | FP&A, sales forecast calls |
| C6 | **Run-rate card** | Value to date, projected year-end at current pace, gap to target, required pace | ○ | 🛠 B6 | Sales, savings programmes, capex spend |
| C7 | **Multi-KPI card** | 2–4 related KPIs in one tile, each with its comparisons and mini chart | ○ (commercial examples exist) | 🛠 B6 | Executive cockpit tiles: Revenue / GM % / EBIT / Cash in one |
| C8 | Bullet list card | 4–10 KPIs as bullets on aligned scales | ◐ (one native bullet per KPI) | 🛠 B6 | Working-capital days, service levels |
| C9 | **Status / exception card** | Count of items by status (e.g. 12 entities: 9 green, 2 amber, 1 red) + top offenders | ○ | 🛠 B13 | Close cockpit, IC reconciliation, supplier OTIF alerts |
| C10 | Leaderboard card | Top N with in-row bars and rank change arrows | ◐ (table + in-cell bars + Top N) | 🛠 B7 | Sales reps, products, branches |
| C11 | Funnel card | Stage values and conversion % | ● funnel | ✅ | Pipeline, recruiting, collections |
| C12 | Narrative card | One generated sentence ("Revenue 42.7 m, +7.8 % vs PY, driven by Region East") | ◐ (dynamic text, Smart Insights) | 🛠 B6 option | Executive summary line, mobile |
| C13 | Gauge card | Dial vs target | ● gauge (QRC3 2026) | ✅ | Avoid; use C4 unless the audience insists |

### 5.3 Which card for which KPI

| KPI behaviour | Card |
|---|---|
| Flow measure with a target (revenue, cost, savings) | C4 progress or C5 landing |
| Flow measure monitored over time (orders, volume) | C3 trend |
| Stock / point-in-time measure (cash, headcount, inventory) | C3 trend with *last value* headline (`valueMode = last`) |
| Ratio (margin %, OTIF %, attrition %) | C2/C3 with variance in **percentage points**, not % of % |
| Count of exceptions (overdue tasks, unmatched IC pairs) | C9 status card |
| Several KPIs that belong together (P&L lines) | C7 multi-KPI or a small IBCS table |
| Lower-is-better (DSO, cost, defects, lead time) | Any card with `increaseIsGood = false` and reversed bullet direction |

---

## 6. Features and interactions beyond native

| Feature | Native | Custom approach | Kit |
|---|---|---|---|
| Styling individual data points by rule (beyond thresholds), e.g. highlight the selected entity in every chart | ◐ (story CSS cannot target single data points) | Widget property `highlightMembers`; script sets it on all widgets | 🛠 B1/B2/B3 common property |
| **Shared scale across several charts** (IBCS) | ◐ (manual axis min/max) | `scaleGroup` + script that reads each widget's max and pushes the group max | 🛠 B1 |
| Cross-highlight without filtering (hover a member, every widget dims the others) | ○ | Widgets fire `onHover`; story script relays to other widgets' `setHighlight()` | 🛠 common |
| Brush / range select on a time axis → filter the page to that range | ◐ (zoom, no range-to-filter) | ECharts `dataZoom`/brush → `setFilters` with the member list | 🛠 B9/B11 option |
| Expand a tree node by node (decomposition) | ○ | Cached levels, re-binding per node, breadcrumbs | 🛠 B5 |
| Client-side what-if (price +2 % → bridge redraws, nothing written back) | ◐ (planning + VDT, or sliders feeding calculated measures) | Slider properties applied inside the widget | 🛠 B15 |
| Annotations and numbered comments on the visual | ◐ (data-point comments are not available for custom widgets) | Annotation JSON property or an annotation measure/attribute from the model | 🛠 B1/B4 option |
| Export the visual as image/PDF/PPT with the story | ● for native; ◐ for custom | `serializeCustomWidgetToImage` | ✅ (bridge, chart) |
| Formatted Excel export of a styled grid | ◐ (native export loses formatting) | Kit XLSX writer | ✅ Export Table |
| Keyboard and screen-reader access to chart marks | ◐ | Focusable marks, `aria-label` sentences, data table fallback | 🛠 all new builds |
| Light/dark and theme awareness | ● | Built-in `themeInfo` property | ✅ pattern in starter template |
| Responsive card grid (4 across on desktop, 1 column on phone) | ◐ (responsive lanes) | Container queries inside the widget | 🛠 B6 |

---

## 7. Comparison guide

### 7.1 Pick the visual by comparison type

| You compare… | Best | Native? | Second choice | Avoid |
|---|---|---|---|---|
| One KPI vs target | Bullet graph or progress card | ● / ◐ | Comparison card | Gauge (uses space, hides the gap) |
| Actual vs plan for 5–20 items, the **gap** is the story | Dumbbell, sorted by gap | ○ → B2 | Integrated variance bar | Grouped bars (reader must subtract) |
| Actual vs plan, **values and variance** both matter | Integrated variance or multi-tier variance chart | ● / ◐ → B1 | Table with in-cell variance bars/pins | Two pies |
| Two periods, **direction and rank swaps** matter | Slope chart | ◐ → B8 | Dumbbell | Grouped bars |
| Two periods, absolute magnitudes must stay readable | Grouped bars (max 2–3 series) | ● | Dumbbell with value labels | 3D, stacked bars |
| Many periods | Line with PY dashed, Δ columns below (multi-tier) | ◐ → B1 | Combination chart | Dual axis with unrelated measures |
| Rank changes over many periods | Bump chart | ◐ → B8 | Small multiples | Spaghetti line chart |
| How we got from A to B | Waterfall / bridge (variance or PVM) | ● / ✅ | Multi-tier variance chart | Stacked bars |
| Composition in two periods | Two 100 % stacked bars side by side | ● | Slope per component | Two pies |
| Many items × many measures | Heat table (thresholds) or small multiples with shared scale | ● / ◐ | Micro-chart table (B7) | Radar with > 6 axes |
| Two groups' distributions | Back-to-back histogram / pyramid | ○ → B12 | Box plots side by side | Overlapping histograms |
| Item vs peer group | Dot on a peer range band (min–max, median tick) | ○ → B2 range mode | Box plot with highlighted point | Bar of the item alone |
| Scenarios (base, best, worst) | Fan / range band | ◐ → ⚙ | Lines for each version | Clustered bars per scenario |
| Sensitivity of a result to drivers | Tornado | ○ → B12 | Sorted variance bars | Table only |

### 7.2 Dumbbell vs slope vs grouped bars

Three charts for "two values per category", three different messages:

| Chart | Encodes the change as | Best when | Limits |
|---|---|---|---|
| Dumbbell | Length of the connecting bar | The size of the gap is the message; 5–20 categories; sort by gap | Only two points per category; never on a dual axis |
| Slope | Tilt of a line, crossings = rank swaps | Direction and rank changes matter more than gap size | Gap size is harder to compare across categories |
| Grouped bars | Two adjacent lengths from zero | Absolute values must be read precisely | The change is implicit; with many groups it becomes hard to read |

### 7.3 Comparison rules that apply to every visual

- **Order**: PY, AC, PL/BU, FC in the same order everywhere; time runs left to right; categories run top to bottom.
- **Notation**: solid = actual, grey = previous period, outline = plan, hatched = forecast (native IBCS templates
  give you this; custom widgets must copy it, B1 exposes it as properties).
- **Variance colour**: green favourable, red unfavourable, decided by `increaseIsGood`, not by the sign.
- **Same scale for the same unit** on one page (B1 `scaleGroup`).
- **Ratios**: variance in percentage points (pp), labelled as such.
- **Label the reference in the title** ("ΔPL", "vs PY YTD"), not only in the legend.
- **Sort by the comparison** (largest gap first) unless the category has a natural order (time, P&L structure).

---

## 8. Playbooks per business area

Each table lists the main questions, the visual to use, its native status and the Datasphere modelling note that
makes it work on an analytic model.

### 8.1 Finance and FP&A

| Question | Visual | Native | Custom | Datasphere note |
|---|---|---|---|---|
| Are we on budget this month/YTD? | C4 progress cards in a row (Revenue, GM, EBIT, Opex) | ◐ | ✅ KPI Card | AC, PL, PY as restricted measures on version/category; YTD via period mapping (see [07](07_Datasphere_Analytic_Model_Recipes.md)) |
| Where will we land? | C5 landing card; fan band for scenarios | ○ | 🛠 B6, ⚙ | Actuals up to the closed period + forecast for the rest: one "AC+FC" measure with a period-based restriction |
| What explains EBIT vs plan? | Variance bridge or PVM bridge | ● / ✅ | ✅ Waterfall/Bridge | PVM needs price and volume measures at a common grain ([07](07_Datasphere_Analytic_Model_Recipes.md)) |
| Which lines/cost centres deviate most? | Multi-tier variance chart; dumbbell for top 15 | ◐ / ○ | 🛠 B1, B2 | Sort by ΔPL in the widget; Top N in the binding |
| Where does the variance come from? | Decomposition tree (entity → cost centre → account) | ◐ | 🛠 B5 | Hierarchies as parent-child dimensions; each node expansion = new filter state |
| P&L statement with variances | IBCS financial table with in-cell variance bars/pins; formatted Excel | ◐ | ✅ Export Table, 🛠 B7 | Account hierarchy with sign handling in the model (revenue positive, costs negative or flipped in the view) |

### 8.2 Group reporting and consolidation

| Question | Visual | Native | Custom | Datasphere note |
|---|---|---|---|---|
| Entity performance on one page | Small multiples with a **shared scale** | ◐ | 🛠 B1 `scaleGroup` | Group currency measure; consolidation unit and group as dimensions |
| How do intercompany flows connect entities? | Chord (entity ↔ partner entity, ribbon = IC volume) | ○ | 🛠 B10 | Partner unit must use the same master data as the entity; one measure, two dimensions |
| Which IC pairs do not reconcile? | Status card (C9) + heat table entity × partner (difference coloured) | ◐ | 🛠 B13 | Difference = receivable − payable on the matched pair; tolerance as a threshold |
| What changed from local to group figures? | Bridge: local → reclass → eliminations → FX → group | ● / ✅ | ✅ | Measures per consolidation step (document type / posting level) |
| Group structure and ownership | Entity tree with ownership %, method, currency | ○ | ⚙ tree / 🛠 B5 layout | Parent-child with attributes; ownership as a measure or attribute |
| Is the close on track? | Close calendar Gantt (planned vs actual task dates, today line) + C9 status card | ○ | 🛠 B4, B13 | Task list with planned/actual start/end as attributes or measures (dates as numbers) |

### 8.3 Treasury, cash and working capital

| Question | Visual | Native | Custom | Datasphere note |
|---|---|---|---|---|
| Cash position and outlook | C3 trend card (last value) + fan band (base/best/worst) | ◐ | ✅, ⚙ | Cash is a stock: last value over time, not a sum of months |
| Cash movement for the period | Cash bridge (opening → operating → investing → financing → closing) | ● / ✅ | ✅ | Cash-flow category as a dimension; opening/closing as stock measures |
| DSO, DPO, DIO vs target | Bullet list card (C8) | ◐ | 🛠 B6 | Lower is better for DSO/DIO: reversed bullets |
| Receivables at risk | Aging bars (buckets) + customer dot plot or Pareto | ● / ✅ | ✅ Pareto | Buckets derived from due date vs key date in an SQL view |
| Daily collection pattern | Calendar heatmap | ○ | 🛠 B11 | Day-level date dimension in the binding |

### 8.4 Sales and commercial

| Question | Visual | Native | Custom | Datasphere note |
|---|---|---|---|---|
| Quota attainment by rep/region | Bullet list or dumbbell (AC vs quota), sorted by gap | ◐ / ○ | 🛠 B6, B2 | Quota at rep/month grain; restrict versions |
| Will we hit the year? | Run-rate card (C6) | ○ | 🛠 B6 | Working-day calendar improves pace calculation |
| Price, volume, mix effects | PVM bridge | ✅ | ✅ | Price and volume at product × customer grain |
| Pipeline health and drop-off | Funnel + pipeline waterfall (new, won, lost, slipped) | ● | ✅ | Pipeline snapshots by week; movement categories from snapshot comparison |
| Share change vs competitors | Slope chart (PY vs AC share) | ◐ | 🛠 B8 | Share as a calculated measure; compare two periods |
| Customer segmentation | Quadrant (value vs growth, size = revenue) | ◐ | ⚙ / 🛠 B3 | Growth as calculated measure; medians as dividers |

### 8.5 Procurement

| Question | Visual | Native | Custom | Datasphere note |
|---|---|---|---|---|
| Where does spend concentrate? | Pareto (cumulative % with 80/95 % lines) | ◐ | ✅ Pareto | Spend at supplier grain; sort in widget |
| How do we source each category? | **Kraljic matrix** (supply risk vs profit impact, quadrants: strategic, leverage, bottleneck, routine; size = spend) | ◐ | ⚙ / 🛠 B3 | Risk and impact scores per category as measures (from a scoring table) |
| Supplier performance | Supplier scorecard table with in-row bullets/sparklines; status card for alerts | ◐ | 🛠 B7, B13 | OTIF, quality, lead time per supplier per month |
| Savings vs target | C4 progress / C6 run-rate | ◐ | 🛠 B6 | Savings baseline as a separate measure |
| Contracted vs actual price | Dumbbell (contract vs paid per material) | ○ | 🛠 B2 | Contract price as attribute/measure from contract data |

### 8.6 Supply chain and inventory

| Question | Visual | Native | Custom | Datasphere note |
|---|---|---|---|---|
| Are we delivering on time and in full? | C3 trend card + OTIF driver tree (on-time × in-full, then carrier, warehouse, supplier) | ◐ | 🛠 B5 | Order-line grain with flags; rates as calculated measures |
| Is lead time stable? | Control chart (mean, UCL/LCL, violations) | ◐ | ⚙ / 🛠 B9 | UCL/LCL as calculated measures (mean ± 3σ over a reference window) |
| Inventory health | ABC Pareto; aging bars; days-of-supply bullet list | ◐ | ✅, 🛠 B6 | Inventory is a stock: snapshot fact, last value over time |
| Where in the warehouse is the problem? | Floor-plan map with bay status | ○ | 🛠 B14 | Location attribute that matches SVG shape ids |
| Demand vs supply gap over coming weeks | Surplus/deficit area chart | ○ | ⚙ / 🛠 B1 option | Weekly buckets for both measures |

### 8.7 HR and people

| Question | Visual | Native | Custom | Datasphere note |
|---|---|---|---|---|
| Headcount and its movement | C3 trend card (last value) + headcount bridge (opening → hires → leavers → transfers → closing) | ◐ / ✅ | ✅ | Stock (headcount) vs flow (hires, leavers) facts; attrition rate on **average** headcount |
| Workforce structure | Population pyramid (age or tenure by gender) | ○ | 🛠 B12 | Bands as derived dimensions |
| Where do people move? | Sankey (grade/department transitions) | ● | ✅ | Start and end state per employee per period |
| Org structure and span of control | Org chart with span and headcount per node | ○ | ⚙ tree / 🛠 B5 | Manager hierarchy as parent-child; privacy: aggregate, avoid names where not needed |
| Engagement survey | Diverging stacked bars | ◐ | 🛠 B12 | Answer scale as dimension with a neutral member |
| Absence pattern | Calendar heatmap | ○ | 🛠 B11 | Day-level absence counts |

### 8.8 Manufacturing, operations and quality

| Question | Visual | Native | Custom | Datasphere note |
|---|---|---|---|---|
| OEE and its losses | Bridge: planned time → availability loss → performance loss → quality loss → OEE | ● / ✅ | ✅ | Loss categories as measures |
| Is the process in control? | Control chart | ◐ | 🛠 B9 | Sample grain (batch/shift) |
| Which line/machine is down now? | Floor plan with status | ○ | 🛠 B14 | Equipment id = SVG shape id |
| Downtime pattern | Calendar heatmap (day) or heatmap (hour × weekday) | ○ / ● | 🛠 B11 / ✅ | Time-of-day dimension for the hourly view |
| Defect causes | Pareto | ◐ | ✅ | Defect code dimension |

### 8.9 Projects and capex

| Question | Visual | Native | Custom | Datasphere note |
|---|---|---|---|---|
| Schedule vs baseline | Gantt with baseline bars, % complete, today line, milestones | ○ | 🛠 B4 | Start/end dates as measures (date as number) or attributes; WBS as hierarchy |
| Budget vs actual vs committed | Bullet list per project; multi-tier variance | ◐ | 🛠 B6, B1 | Commitment as a separate measure |
| Portfolio risk vs value | Quadrant (risk vs NPV, size = budget) | ◐ | ⚙ / 🛠 B3 | Scores from a project master |
| Spend curve (S-curve) vs plan | Cumulative line actual vs plan | ◐ | ⚙ | Cumulative measures or running sum in widget |

### 8.10 Executive cockpit

| Element | Visual | Notes |
|---|---|---|
| Header | 4–6 C7 multi-KPI or C3 trend cards | One comparison window for the whole row |
| Story of the period | Narrative line (C12) + one bridge | Generated text from the same bound data as the cards |
| Exceptions | C9 status cards | Click opens the exception list |
| Detail on demand | Popup or page per card | Linked analysis from the card's sparkline/point |

---

## 9. Build backlog for the kit

Effort: S ≈ 1–2 days, M ≈ 3–5 days, L ≈ 1–2 weeks (including harness tests and documentation, as for the existing
widgets). "Base" is the kit module to start from.

| ID | Widget | Binding shape (Datasphere analytic model) | Linked analysis | Base | Effort | Priority |
|---|---|---|---|---|---|---|
| B1 | **IBCS Variance Chart**: multi-tier (AC/PL bars, ΔPL abs bars, ΔPL % pins), scenario notation properties, `scaleGroup`, surplus/deficit and range-band modes, annotations | 1 category dim (time or structure) + measures AC, reference (PL/PY/FC) [+ 2nd reference, + low/high band] | Category member | Waterfall/Bridge (SVG renderer, shared formatter) | L | P1 |
| B2 | **Dumbbell / Range Dot Plot**: two values per category, sort by gap, colour by favourability, optional peer band (min/max/median) | 1 dim + 2 measures (+ 3 for the band) | Category member | Universal Chart (ECharts custom series) or plain SVG | M | P1 |
| B3 | **Quadrant Matrix**: scatter with dividers (fixed, median or target), shaded and named zones, bubble size, label top N | 1 point dim + x, y (+ size) measures (+ colour dim) | Point member | Universal Chart scatter + `markArea`/`markLine` | S–M | P1 |
| B4 | **Gantt / Timeline**: plan vs baseline bars, % complete, milestones, today line, WBS hierarchy, status colours | Task dim (hierarchy optional) + start, end (+ baseline start/end, % complete) as numeric date measures | Task member | New (SVG) | L | P1 |
| B5 | **Decomposition Tree**: node-by-node split of one measure by chosen dimensions, variance mode (AC vs PL per node), formula mode (driver tree) | 1 measure (+ reference) + up to 5 dims, levels fetched on expand | Node path as multi-dimension filter | New (SVG) | L | P1 |
| B6 | **KPI Card v2**: modes trend, progress, bullet, landing, run-rate, multi-KPI (2–4), bullet list; narrative line; responsive grid; full `aria-label` | Value, comparison, target, forecast measures (+ trend dim) | Sparkline point / card click event | KPI Card | M | P1 |
| B7 | **Micro-chart Table**: columns as sparkline, bullet, variance bar, variance pin, rank change; IBCS headers; formatted Excel export | Row dim (hierarchy) + measures + optional trend dim | Row member | Export Table | M–L | P1 |
| B8 | Slope / Bump chart | 1 item dim + period dim (2 or n periods) + 1 measure (or rank) | Item member | Universal Chart (line) | S | P2 |
| B9 | Control Chart: mean, UCL/LCL (bound or computed), Western Electric / Nelson rule markers, brush-to-filter | Time/sample dim + value measure (+ UCL/LCL measures) | Sample member or range | Universal Chart (line + markers) | M | P2 |
| B10 | Chord: flows between members of one set | Source dim + partner dim (same master) + 1 measure | Pair (source, partner) | New (D3-chord, ~10 KB) or ECharts graph circular | M | P2 |
| B11 | Calendar Heatmap | Day-level date dim + 1 measure | Day or range | Universal Chart (ECharts calendar) | S | P2 |
| B12 | Butterfly / Tornado / Pyramid / Diverging stacked | 1 category dim + 2 measures (or 1 measure + 2-member split dim) | Category member | Universal Chart (mirrored bars) | S | P2 |
| B13 | Status / Exception Board | Item dim + status measure or thresholds + value | Item member, status group | KPI Card styles | S–M | P2 |
| B14 | Floor Plan / SVG Shape Map | Location dim (member id = shape id) + status/value measure; SVG as property | Location member | New (inline SVG) | M | P2 |
| B15 | What-if Bridge: sliders for price/volume/FX applied to a bound bridge, no write-back | Bridge binding + driver properties | As bridge | Waterfall/Bridge | M | P3 |

Shared requirements for every new build (same standard as the current kit):

- Data binding with auto builder panel, linked analysis via `setFilters`/`removeFilters`, Escape clears the
  selection (as in the bridge and the table today).
- `serializeCustomWidgetToImage` for PDF/PPT export, `supportsLWV` decided per widget, `themeInfo` aware.
- Number formatting through the shared formatter (scale, decimals, locale, sign for variances, pp for ratios).
- `increaseIsGood`, scenario notation and palette as properties; `highlightMembers` and `onHover` for cross-widget
  highlighting.
- Keyboard-focusable marks and `aria-label` sentences; a hidden data table for screen readers.
- Harness test file with at least: render, empty data, linked-analysis click, export image, property round trip.

---

## 10. Recipes you can use today with the current kit

These need no new code, only configuration.

| Need | Widget and setting |
|---|---|
| Pareto with 80 % and 95 % lines (ABC classes) | Universal Chart `chartType = pareto` (series 0 = bars, series 1 = cumulative share as 0–1, already with an 80 % line); `optionOverride`: `{"series":[{},{"markLine":{"data":[{"yAxis":0.8},{"yAxis":0.95,"label":{"formatter":"95%"}}]}}]}` |
| Quadrant matrix (simple) | Universal Chart `scatter`; `optionOverride` with `series[0].markLine` for the two dividers (`xAxis`/`yAxis` values) and `series[0].markArea` for shaded zones; give positions in data units (`"min"`/`"max"` in a mark area refer to the series' own data, not the axis), and use B3 when dividers must follow medians or targets |
| Control chart (simple) | Universal Chart `line`; bind value, UCL, mean and LCL as four measures from the model; `optionOverride` `{"series":[{},{"lineStyle":{"type":"dashed"}},{"lineStyle":{"type":"dotted"}},{"lineStyle":{"type":"dashed"}}]}` |
| Range band / fan | Universal Chart `area`; bind measures in the order low, (high − low), base; `optionOverride` `{"series":[{"stack":"band","areaStyle":{"opacity":0},"lineStyle":{"opacity":0}},{"stack":"band","areaStyle":{"opacity":0.25},"lineStyle":{"opacity":0}},{"areaStyle":{"opacity":0}}]}` |
| Sunburst cost structure | Universal Chart `sunburst` on a parent-child account or cost-centre hierarchy |
| Headcount, cash or OEE bridge | Waterfall/Bridge in measure mode with opening, movements and closing measures |
| Trend card on a stock measure | KPI Card `valueMode = last`, comparison = PY measure, `showCompareLine = true` |

`optionOverride` is merged into the generated ECharts option element by element, so `series[0]` in the override
changes the first generated series. It changes styling and adds marks; forms that need a different data mapping
(dumbbell, Gantt, chord, calendar) need their own code (section 9).

---

## 11. Verify before you build

1. **Native check**: open "What's New" for your tenant release and the chart picker; new native types (like Sankey,
   gauge and funnel in QRC3 2026) remove the need for a custom version.
2. **BDC check**: on Business Data Cloud, look in the Custom Widget Library for the same visual (dumbbell, org
   chart, tornado were announced).
3. **Composite check**: can a composite of native widgets do it with acceptable performance?
4. **Add-on check**: is it only a tooltip or an overlay on a supported native chart?
5. **Data check**: can the analytic model deliver the binding shape in section 9 (restricted measures, date as
   number, hierarchy as parent-child)? Run the POC items in [06](06_Deployment_Testing_Troubleshooting.md).
6. **Audience check**: will the readers understand it without training? If not, plan a one-line "how to read"
   caption or choose the second-choice visual from section 7.
7. **Licence check** for any library (ECharts Apache-2.0 is fine; Highcharts, amCharts, DHTMLX Gantt need
   commercial licences for enterprise use).

---

## 12. Sources

Native SAC and SAP guidance:

- SAP Analytics Cloud release highlights (My Metrics, Horizon themes): https://www.sap.com/products/data-cloud/cloud-analytics/features/release-highlights.html
- SAC dashboard design best practices, IBCS section (SAC is IBCS certified; fill patterns; variance rules): https://www.sap.com/design-system/sac-dashboard-design-best-practices/ibcs
- SAP Learning, Working with IBCS in SAP Analytics Cloud: https://learning.sap.com/courses/designing-stories-in-sap-analytics-cloud/working-with-ibcs-standards-in-sap-analytics-cloud
- Visualizing data with SAP Analytics Cloud (chart categories, variance waterfall, in-cell deviation): https://community.sap.com/t5/technology-blogs-by-sap/visualizing-data-with-sap-analytics-cloud/ba-p/14033467
- Composite data source configuration (QRC2 2024): https://blogs.sap.com/t5/technology-blogs-by-sap/composite-data-source-configuration-in-optimized-story-experience/ba-p/13678967
- SAC highlight: composite (QRC4 2023): https://community.sap.com/t5/technology-blogs-by-sap/sap-analytics-cloud-highlight-composite/ba-p/13579369
- KBA 3778420, copy/paste of composites (QRC3 2026): https://userapps.support.sap.com/sap/support/knowledge/en/3778420
- SAC Q4 2025 new features (composites, variance scripting, My Metrics), ZPARTNER: https://www.zpartner.eu/?p=13224
- In-cell charts and variance bar/pin: https://www.zpartner.eu/?p=2319
- New chart types Sankey, gauge, funnel and the Highcharts 12 engine: see the QRC3 2026 posts listed in [08 — Resource Library](08_Resource_Library.md)
- NextLytics, Smart Tile custom widget (multi-KPI tile) and SAC wish list (numeric point performance, CSS on data points): https://nextlytics.com/blog/smart-tiles-sap-analytics-cloud , https://nextlytics.com/blog/sap-analytics-cloud-features-our-wish-list-for-2023

Visualisation and design standards:

- Financial Times Visual Vocabulary (category definitions): https://github.com/ft-interactive/visual-vocabulary
- IBCS standards (SUCCESS, version 2.0 aligned with ISO 24896): https://www.ibcs.com/IBCS/ , https://www.ibcs.com/de/standards
- Zebra BI, IBCS and CFO dashboard notes: https://zebrabi.com/ibcs/ , https://zebrabi.com/cfo-dashboard/
- Stephen Few, bullet graph design specification (summary): https://betterevaluation.org/node/1784
- Bullet chart anatomy and accessibility notes: https://www.visualizing.org/bullet-chart
- Dumbbell chart guidance: https://domo-webflow.domo.com/learn/charts/dumbbell-plot-chart , https://www.visualizing.org/dumbbell-chart
- Slope chart guidance: https://domo-webflow.domo.com/learn/charts/slope-chart
- Grouped bars vs dumbbell (EU data visualisation guide): https://data.europa.eu/apps/data-visualisation-guide/grouped-bars
- KPI card design (context, comparisons): https://tabulareditor.com/zh-cn/blog/kpi-card-best-practices-dashboard-design , https://thedataschool.co.uk/mila-kholodiy/power-bi-kpis-choosing-the-right-kpi-visual/

Area playbooks:

- Supply chain dashboards (inventory Pareto, supplier scorecards): https://ecosire.com/blog/power-bi-supply-chain-logistics-dashboard
- HR analytics (stock vs flow, average headcount for attrition): https://ecosire.com/blog/power-bi-hr-people-analytics-dashboard
- Kraljic matrix from a spend cube: https://www.payflows.io/blog/step-by-step-guide-to-creating-and-using-a-kraljic-matrix-in-procurement
- Pipeline waterfall analysis: https://forecastio.ai/blog/sales-pipeline-dashboard
- CFO dashboards (bridges, working capital bullets): https://fanruan.com/en/blog/visual-financial-reporting-for-cfos , https://corpready.in/articles/263-excel-dashboard-creation

Next: back to the [README](../README.md) or the [Possibilities Catalog](05_Possibilities_Catalog.md).
