---
name: html-report
description: Generate a rich, interactive, mobile-first HTML report instead of markdown and return a browser-ready private or public URL. Use when the user asks for a report, writeup, summary page, dashboard, share link, or any output that benefits from charts, sortable tables, tabs, search, diagrams, or visual polish. Save to the configured private report server by default; publish to GitHub Pages only when explicitly requested.
---

# html-report

Generate a self-contained, polished, **interactive** HTML report. Save it to the user's private report server by default; publish it to GitHub Pages only when explicitly requested.

## Why this skill exists (read this first)

Codex often returns markdown by default. Markdown is fine for plain text, but it's a **failure mode** for any output that has data, comparisons, code, or detail that the reader will want to navigate. This skill exists to push you toward HTML's strengths. If you generate a wall of `<p>` tags and `<h2>` headings, **you have wasted this skill** — markdown would have done the same job.

Concretely, HTML can do all of these and markdown cannot:

| Capability | Use it for |
|---|---|
| **Interactive charts** (Chart.js / Plotly / ApexCharts) | any numeric data, trends, distributions |
| **Sortable / filterable / searchable tables** (DataTables, grid.js) | any tabular data with >5 rows |
| **Tabs** | comparing alternatives, before/after, multiple views of same data |
| **Collapsible `<details>` sections** | proofs, raw output, optional deep-dives, long appendices |
| **Syntax-highlighted code with copy buttons** (highlight.js / Prism) | any code snippet |
| **Mermaid / Graphviz diagrams** | architecture, flows, state machines, dependency graphs |
| **KaTeX / MathJax** | any equation |
| **Hover tooltips, popovers, modals** | term definitions, footnotes, citations |
| **Animations / transitions** | state changes, reveals, focus shifts |
| **Dark mode toggle** | every report |
| **Sticky table-of-contents with scroll-spy** | any report >2 sections |
| **Responsive layout** | mobile reading |
| **Print stylesheet** (`@media print`) | shareable PDFs |
| **Embedded video / audio** | media-heavy work |
| **Search / filter inputs** over content | long lists, FAQs, glossaries |

Every report you produce should use **at least three** of these. If you can't find three that apply, the user probably wanted markdown — push back and ask.

## Configuration

Read `config.json`, then apply any values from the ignored `config.local.json` beside it. Fields:

- `output_dir` — private report directory; default `~/Documents/html-reports`.
- `private_base_url` — Meshnet report-server URL. Return this URL plus the filename.
- `validation_viewports` — required mobile and desktop render sizes in CSS pixels. Raw display pixels are not CSS pixels.
- `local_repo_path`, `reports_dir`, `base_url` — existing GitHub Pages publishing settings.

## Workflow

1. **Plan the report.** Before writing HTML, decide:
   - What's the headline finding? (Goes at the top, big.)
   - What sections does it have?
   - For each section, which interactive element fits? (data → chart, comparison → tabs, code → highlighted block, long → collapsible, etc.)
   - Will it have a TOC? Use a sticky TOC on desktop when useful. On mobile, omit it or use one collapsed native disclosure; never use a horizontally scrolling row of navigation pills.

2. **Generate ONE self-contained HTML file** at `<output_dir>/<YYYY-MM-DD>-<slug>.html`.
   - Slug should be short, kebab-case, descriptive (`gpt5-vs-claude-benchmark`, not `report-1`).
   - Start from `templates/base.html` — it already wires up Tailwind, highlight.js, Chart.js, Mermaid, KaTeX, dark mode, sticky TOC, and print styles.
   - All libraries via CDN (no build step).
   - Custom CSS in a single `<style>` block; custom JS in a single `<script>` block at the bottom.
   - Set `<title>`, `<meta name="description">`, `<meta name="viewport">`, and Open Graph tags (the report will be shared as a link — OG tags make link previews work).

3. **Validate.** Check JavaScript syntax and render the same file at every configured viewport. Confirm no page-level horizontal scrolling, clipped controls, illegible chart labels, or hover-only interactions.

4. **Return the private URL.** Join `private_base_url` and the filename. Verify it responds before returning it; do not fabricate a URL.

5. **Publish publicly only when explicitly requested.** Copy the report into `<repo>/<reports_dir>/`, then run:
   ```bash
   python3 <skill-dir>/scripts/publish.py <html-path-relative-to-repo> "<title>" "<one-line description>"
   ```
   The script:
   - Updates `reports/manifest.json` (which the index page reads).
   - `git add -A`, `git commit -m "report: <title>"`, `git push`.
   - Prints the public URL.

Return the publish script's URL verbatim.

## Quality bar — required for every report

- **Single file**, no build step, CDN-only deps.
- **Mobile-responsive** — test in your head: does it work on a 375px viewport?
- **Mobile-first** — validate from 360–520 CSS px wide and at the configured mobile viewport. Use a 16px minimum body font, 44px touch targets, single-column primary flow, horizontally scrollable table wrappers, and charts that remain legible without hover.
- **Desktop-complete** — validate at the configured desktop viewport and progressively enhance spacing, navigation, and multi-column layout without changing content or URL.
- **One responsive artifact** — serve identical HTML to every device. Do not generate separate phone/desktop editions or rely on IP, User-Agent, or server-injected device styles.
- **Dark mode** with a toggle, persisted to localStorage.
- **Title + description + OG tags** in `<head>`.
- **Prefers-reduced-motion respected** — no jarring animations for users who opted out.
- **Adaptive navigation** — use a sticky TOC for long desktop reports; omit it or use a collapsed native disclosure on mobile.
- **Anchor links** on all section headings (`id="..."`).
- **No inline `style=""`** — use a single `<style>` block.
- **No hardcoded light-mode colors** — use Tailwind's `dark:` variants or CSS variables.

## Strongly recommended (pick what fits the content)

- **Tables → DataTables.js** (sort, filter, paginate). Drop-in: load `datatables.net` CDN, wrap with `$('table').DataTable()`.
- **Numeric data → Chart.js** (bar/line/pie/scatter/radar). Always set `responsive: true, maintainAspectRatio: false`.
- **Dense chart axes → horizontal scroll.** For more than six X-axis categories on mobile, put a legible minimum-width chart inside an overflow wrapper instead of compressing labels.
- **Wide tables → horizontal scroll.** Give the table a content-driven minimum width inside an overflow wrapper; never squeeze columns until body text wraps word-by-word.
- **Code → highlight.js**, plus a "copy" button per block.
- **Comparisons (3+ alternatives) → tabs**. Hand-roll with `<input type="radio">` + CSS or use the snippet in `templates/base.html`.
- **Long detail → `<details>` collapsibles** with descriptive `<summary>`.
- **Architecture / flow → Mermaid**. `<pre class="mermaid">graph TD; A-->B</pre>`.
- **Math → KaTeX** auto-render (faster than MathJax). Inline: `$x^2$`. Display: `$$\int...$$`.
- **Multiple views of same data → segmented control / toggle**.
- **Long lists (>20 items) → search/filter input** at the top.
- **Citations / footnotes → popovers** that show the source on hover, not at the bottom of the page.

## Anti-patterns (do not ship these)

- ❌ Markdown disguised as HTML — paragraphs and bullets only.
- ❌ Static images of charts when the data is small enough to render live.
- ❌ Tables of >10 rows with no sort or filter.
- ❌ Walls of code with no syntax highlighting.
- ❌ 5MB of JS for a 200-word report. Pick libraries that fit the content.
- ❌ Light-only color choices.
- ❌ "Click here" links — use descriptive link text.
- ❌ Charts that overflow on mobile.
- ❌ Horizontally scrolling pill rows for a mobile table of contents.
- ❌ Shrinking dense charts or wide tables to viewport width until labels become unreadable.
- ❌ Designing at the phone's 1440 physical pixels. Browser layout uses CSS pixels and varies with Android display scaling.
- ❌ Device-specific HTML files, URLs, request detection, or server-side style injection.

## Example: planning a "model benchmark" report

| Element | Choice |
|---|---|
| Headline number | Big number at the top, with sparkline trend |
| Per-task results | DataTable, sortable by score |
| Two models compared head-to-head | Bar chart |
| Three configs compared | Tabs |
| Per-task example outputs | Collapsibles |
| Methodology | Collapsible at the bottom |
| Math (eg. ELO formula) | KaTeX |

## Draft mode

Private reports are never committed or pushed. If a public GitHub Pages report is requested as a draft, pass `--draft` to the publish script.

## After publishing

Return the verified private URL or the public URL printed by the publishing script. Surface any failed validation, server request, commit, or push.

## When NOT to use this skill

- One-line answers.
- Throwaway debug output.
- Anything the user explicitly asked for as markdown / plain text.
- Code that goes into a file in the user's project.
