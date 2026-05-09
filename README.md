# Anthropic Shipped Outcomes. The Real Story Is Verification Becoming a SKU.

A deep-dive analysis of Anthropic's May 6, 2026 release: Outcomes, Dreams, Multi-Agent Orchestration, and Webhooks. What they are, why they shipped together, and what builders should stop writing.

**~3,500 words | ~14 minute read | 11 interactive diagrams**

## Article + Diagrams

- [article.md](article.md) — Full article text with `[DIAGRAM N]` placeholders
- [Outcomes Diagrams.html](Outcomes%20Diagrams.html) — All 11 diagrams (requires internet for Google Fonts)
- [diagrams.html](diagrams.html) — Standalone version with fonts inlined (fully offline)

Each diagram is a self-contained card designed for screenshot capture at Medium's 760px reading width. Scroll-reveal animations play on first view; print and `prefers-reduced-motion` disable them automatically.

## Style Comparison (earlier iterations)

- **`index.html`** — Cozypet system. IBM Plex Sans + Mono. Anthropic-aligned palette. 11 diagrams.
- **`editorial.html`** — Editorial direction. Fraunces serif + Source Serif body. Magazine layout. 10 diagrams.
- **`article.html`** / **`article-editorial.html`** — Full article rendered in each style.
- **`images/`** — Diagram PNG renders.

### Diagram Index

| # | Title |
|---|-------|
| 1 | Outcomes is a loop, not a feature |
| 2 | The outcome lifecycle, as events |
| 3 | The full lifecycle, with iterations |
| 4 | Same pattern. Different decade. |
| 5 | Three places the eval loop can live |
| 6 | Three primitives. One bus. |
| 7 | Outcomes is a second brain on the same bus |
| 8 | Four harness layers. Four SKUs. |
| 9 | Harness as code, then as product line |
| 10 | Where to start. The adoption ladder. |
| 11 | What you keep. What Anthropic owns. |

## Key Sources

- [New in Claude Managed Agents](https://claude.com/blog/new-in-claude-managed-agents) — Launch announcement (May 6, 2026)
- [Scaling Managed Agents: Decoupling the brain from the hands](https://anthropic.com/engineering/managed-agents) — Architecture rebuild (April 8, 2026)
- [Building Effective Agents](https://anthropic.com/research/building-effective-agents) — Original evaluator-optimizer pattern (December 2024)
