# Shapeshifter — Generative Landing Page Engine

*The page that watches you read it, then rewrites itself.*

Every visitor sees a different page — not from pre-built variants, but **generated** from raw product facts at the moment they arrive. The copy, layout, structure, tone, and visual design are all created based on who's looking.

## Quick Start

Just open `index.html` in a browser. No build step, no dependencies, no npm.

For Netlify: push this repo and it deploys automatically.

## URL Parameters

### Persona simulation
| Parameter | What it simulates |
|-----------|-------------------|
| `?persona=cto` | Technical evaluator — GitHub referrer, desktop, data-driven |
| `?persona=ceo` | Executive buyer — LinkedIn referrer, story-driven |
| `?persona=marketer` | Ad clicker — Google Ads, concise preference |
| `?persona=returning` | 3rd visit, previously viewed pricing — decision stage |
| `?persona=ppc` | PPC ad click — high urgency |
| `?persona=hackernews` | HN referrer — expert, skeptical |
| `?persona=mobile` | iPhone, evening, commute context |

### Modes
| Parameter | Effect |
|-----------|--------|
| `?debug=true` | Show debug panel with persona scores, signals, transforms, mutation log |
| `?split=cto,ceo` | Split-screen comparing 2-3 persona generations side by side |
| `?split=cto,ceo,marketer` | Three-column comparison (tabs on mobile) |

### Combine them
```
?persona=cto&debug=true
?split=cto,ceo&debug=true
```

## How It Works

1. **Signal Collection** — Reads URL params, device type, screen size, time of day, connection speed, referrer, visit history (localStorage)
2. **Persona Engine** — Builds a persona profile: role, intent, engagement, sophistication, urgency, stage, trust level, content preference
3. **Transform Engine** — Generates all copy from raw product facts. Every headline, description, CTA, and section intro is produced by transform functions — not selected from pre-written variants
4. **Page Generator** — Assembles blocks in persona-optimised order with generated content
5. **Mutation Engine** — Monitors behavioural signals every 2 seconds and rewrites unseen sections in real-time

## Behavioural Mutations

The page adapts as you interact:
- **Fast scrolling** → Injects bold stat as pattern interrupt
- **Deep reading** → Upgrades upcoming sections to detailed verbosity
- **CTA hover without click** → Adds trust reassurance
- **Copy event** → Injects shareable content block
- **Rage clicks** → Shows help prompt
- **Scroll reversals** → Regenerates footer CTA with clearer language

## Demo Product

Uses **Versantus AI** (versantus.ai) — a real AI-augmented web agency — as the demo product. All product facts are in `product.json`.

## Files

- `index.html` — Self-contained demo (inline CSS + JS, ~80KB)
- `product.json` — Raw product data (no persona variants)
- `netlify.toml` — Netlify deployment config
- `SPEC.md` — Full specification

## Architecture

Everything runs client-side. No server, no API, no LLM calls. The transform engine uses template-based generation that produces genuinely different copy from the same source data. A CTO and a CEO see completely different pages — different headlines, different feature descriptions, different block ordering, different visual themes.
