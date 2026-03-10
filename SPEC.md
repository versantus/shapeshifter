# Shapeshifter — Generative Landing Page Engine

*The page that watches you read it, then rewrites itself.*

## The Idea

Every visitor to your site is different. A CTO evaluating your platform doesn't care about the same things as a marketing manager. A returning visitor who spent 3 minutes reading your pricing page last week doesn't need the same hero as a first-timer from a Google ad.

Today, we build one landing page and hope it works for everyone. Or we build 5 variants and A/B test. Both are crude.

Shapeshifter takes a product data file (JSON) — one set of raw facts — and **generates** an entire landing page on the fly. Not selects from pre-built variants. *Generates.* The copy, the layout, the structure, the tone, the visual weight — all created at the moment someone arrives, and continuously rewritten as they interact.

No page exists until a visitor shows up. Every page is unique. The same URL produces a completely different experience for a CTO, a marketing manager, and a developer — not because we wrote three pages, but because the engine understands the product and rewrites it for whoever's looking.

## How It Works

```
                    ┌─────────────────┐
                    │  PRODUCT DATA   │
                    │  (JSON)         │
                    │                 │
                    │  • features     │
                    │  • benefits     │
                    │  • testimonials │
                    │  • pricing      │
                    │  • case studies │
                    │  • media        │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
   Signals ────────►│  SHAPESHIFTER   │
                    │  ENGINE         │
   • UTM params    │                 │
   • referrer      │  persona ───┐   │
   • device        │  behaviour ─┤   │
   • time of day   │  product ───┤   │
   • scroll depth  │             ▼   │
   • click pattern │  GENERATOR      │
   • return visit  │  (rules / LLM)  │
   • geo location  │             │   │
                    └────────┬───┘   │
                             │       │
                    ┌────────▼────────┐
                    │  RENDERED PAGE  │
                    │                 │
                    │  Unique layout  │
                    │  Adapted copy   │
                    │  Relevant proof │
                    │  Tuned CTA      │
                    │  Right imagery  │
                    └─────────────────┘
```

## Signals (What We Know)

### Arrival Signals (available immediately)
| Signal | Source | What it tells us |
|--------|--------|-----------------|
| `utm_source` / `utm_medium` / `utm_campaign` | URL params | Where they came from, what ad they clicked |
| `referrer` | HTTP header | LinkedIn? Google? Direct? HackerNews? |
| `device` | User agent | Mobile (scrolling on commute) vs desktop (at work) |
| `geo` | IP geolocation | Country, city, timezone |
| `time_of_day` | Clock | Morning research vs evening browsing |
| `language` | Accept-Language | Browser locale |
| `screen_size` | JS | Not just responsive — layout complexity decisions |
| `connection` | Navigator API | Fast? Slow? Reduce media weight accordingly |
| `return_visit` | Cookie/localStorage | Been here before? What did they see? |
| `previous_pages` | Cookie/localStorage | What they looked at last time |

### Behavioural Signals (gathered in real-time)
| Signal | Detection | What it tells us |
|--------|-----------|-----------------|
| `scroll_velocity` | Scroll events | Skimming fast = bored. Slow = reading carefully |
| `scroll_depth` | Scroll position | How far they got before bouncing |
| `dwell_time` | Per-section timers | Which sections hold attention |
| `hover_patterns` | Mouse tracking | What they're drawn to (desktop) |
| `click_targets` | Click events | What they interact with |
| `rage_clicks` | Rapid repeated clicks | Frustrated — something's broken or unclear |
| `idle_time` | Activity timeout | Tabbed away? Lost interest? |
| `copy_events` | Copy detection | Copying text = sharing with a colleague |
| `scroll_direction` | Scroll reversals | Went back up = re-reading something important |
| `form_hesitation` | Input focus/blur timing | Hovering over the CTA but not clicking |

### Derived Persona (inferred from signals)

The engine builds a live persona profile that updates as the user interacts:

```json
{
  "persona": {
    "role": "technical-evaluator",     // or "executive-buyer", "end-user", "researcher"
    "intent": "comparing-options",      // or "ready-to-buy", "just-browsing", "returning"
    "engagement": "high",               // or "medium", "low", "declining"
    "sophistication": "expert",         // or "intermediate", "beginner"
    "urgency": "medium",                // or "high", "low"
    "stage": "consideration",           // or "awareness", "decision"
    "trust_level": "skeptical",         // or "neutral", "warm"
    "device_context": "desk-research",  // or "commute-scroll", "meeting-check"
    "content_preference": "data-driven" // or "story-driven", "visual", "concise"
  }
}
```

**How roles are inferred:**
- `utm_source=linkedin` + `referrer` contains `/posts/` + desktop → likely professional evaluator
- `utm_campaign=pricing-ad` + direct to pricing → ready to buy
- HackerNews referrer + fast scroll + long dwell on technical sections → technical evaluator
- Mobile + evening + slow scroll → casual browser / researcher
- Return visit + previously viewed case studies + now on pricing → decision stage
- Copied text last visit + returning within 48h → sharing with team, building a case

## Page Components (What Gets Generated)

### Layout Blocks

Each page is assembled from these blocks, ordered and configured per persona:

| Block | Description | Persona influence |
|-------|-------------|-------------------|
| `hero` | Main headline, subhead, CTA, background | Copy tone, urgency, image choice |
| `social_proof` | Testimonials, logos, stats | Which testimonials (technical vs business), prominence |
| `features` | Feature grid or list | Which features highlighted, depth of explanation |
| `benefits` | Outcome-focused cards | Business benefits vs technical benefits |
| `comparison` | Us vs alternatives | Show to evaluators, hide from casual browsers |
| `case_study` | Deep-dive story | Match industry/role to visitor's likely industry |
| `pricing` | Plans/tiers | Prominence, which tier highlighted, annual vs monthly |
| `faq` | Common questions | Technical FAQ vs business FAQ |
| `cta_strip` | Conversion prompt | Soft ("Learn more") vs hard ("Start free trial") |
| `trust` | Security badges, certifications | More prominent for enterprise/finance visitors |
| `team` | People behind the product | Show to relationship-driven personas |
| `technical` | API docs, architecture, specs | Only for technical evaluators |
| `roi` | Calculator or stats | Show to executive buyers |
| `video` | Product demo or testimonial | Only on fast connections, not on commute |
| `urgency` | Limited offer, countdown | Only for high-intent return visitors |

### Adaptive Properties

Every block has properties the engine tunes:

```typescript
interface BlockConfig {
  // Structure
  type: BlockType;
  position: number;          // order in page
  visible: boolean;          // show/hide entirely
  
  // Content selection
  variant: string;           // which content variant to use
  contentIds: string[];      // which testimonials/features/cases to show
  
  // Copy tone
  tone: 'technical' | 'business' | 'casual' | 'urgent';
  verbosity: 'concise' | 'standard' | 'detailed';
  
  // Visual
  layout: 'full-width' | 'contained' | 'split' | 'cards' | 'minimal';
  colorTemperature: 'warm' | 'neutral' | 'cool';   // warm = friendly, cool = professional
  mediaWeight: 'heavy' | 'balanced' | 'light';      // images/video vs text
  whitespace: 'spacious' | 'standard' | 'compact';
  
  // CTA behaviour
  ctaStyle: 'soft' | 'medium' | 'strong';
  ctaText: string;           // generated per persona
  ctaAction: 'demo' | 'trial' | 'contact' | 'download' | 'learn-more';
}
```

## Product Data Format

The input JSON that describes everything Shapeshifter has to work with:

```json
{
  "product": {
    "name": "Acme Platform",
    "tagline": "Ship faster without the chaos",
    "url": "https://acme.io",
    "category": "developer-tools"
  },
  
  "brand": {
    "primaryColor": "#3b82f6",
    "secondaryColor": "#1e293b",
    "fonts": { "heading": "Inter", "body": "Inter" },
    "logo": "/logo.svg",
    "tone": "professional-but-human",
    "values": ["speed", "reliability", "simplicity"]
  },

  "headline": "Deploy in minutes, not meetings",
  "subheadline": "Zero-config CI/CD that just works. Git push → production in 90 seconds.",
  
  "features": [
    {
      "id": "auto-deploy",
      "name": "Auto Deploy",
      "description": "Automatic builds triggered by git push. Supports Docker, Node, Python, Go, Rust. Blue-green deployments with automatic rollback on health check failure.",
      "metric": "90 seconds from push to production",
      "icon": "rocket",
      "category": "core"
    }
    // ... more features — just facts, no persona variants
  ],
  
  "testimonials": [
    {
      "id": "t1",
      "quote": "We reduced deployment time from 45 minutes to 90 seconds. The engineering team actually cheered.",
      "author": "Sarah Chen",
      "role": "VP Engineering",
      "company": "FastTrack",
      "industry": "fintech",
      "metric": "30x faster deploys"
    }
    // ... raw quotes, no persona tagging
  ],
  
  "case_studies": [
    {
      "id": "cs1",
      "company": "FastTrack",
      "industry": "fintech",
      "size": "50-200",
      "challenge": "45-minute deploys blocking feature velocity",
      "outcome": "90-second deploys, 4x more releases per week",
      "metrics": { "deploy_time": "-97%", "release_frequency": "+300%", "incidents": "-60%" }
    }
  ],
  
  "pricing": {
    "model": "tiered",
    "currency": "USD",
    "tiers": [
      { "name": "Starter", "price": 0, "period": "month", "features": ["5 projects", "1 user", "Community support"] },
      { "name": "Pro", "price": 49, "period": "month", "features": ["Unlimited projects", "10 users", "Priority support", "Custom domains"], "highlighted": true },
      { "name": "Enterprise", "price": null, "period": "month", "features": ["Everything in Pro", "SSO/SAML", "SLA", "Dedicated support"] }
    ],
    "annual_discount": 20
  },
  
  "media": {
    "images": [
      { "id": "dashboard", "src": "/images/dashboard.webp", "caption": "Real-time deployment dashboard", "subject": "ui" },
      { "id": "terminal", "src": "/images/terminal.webp", "caption": "CLI deployment flow", "subject": "code" },
      { "id": "team", "src": "/images/team.webp", "caption": "Teams shipping faster", "subject": "people" }
    ],
    "demo_video": "/video/product-demo.mp4"
  },
  
  "competitors": ["Heroku", "Vercel", "Railway", "Render"],
  
  "differentiators": [
    "90-second deploys vs industry average of 15 minutes",
    "Zero configuration required",
    "Full-stack support, not just frontend",
    "Self-host option available"
  ],
  
  "trust": {
    "customer_count": 2400,
    "uptime": "99.99%",
    "deploys_per_day": 50000,
    "certifications": ["SOC2", "GDPR"],
    "logos": ["/logos/fasttrack.svg", "/logos/acme-corp.svg"]
  },
  
  "faq": [
    { "q": "What languages do you support?", "a": "Node.js, Python, Go, Rust, Ruby, Java, .NET, PHP — anything that runs in Docker." },
    { "q": "How long does onboarding take?", "a": "Most teams are live in under an hour. We handle migration from your existing CI/CD." },
    { "q": "Is there a free tier?", "a": "Yes — Starter is free forever for up to 5 projects." },
    { "q": "Can I self-host?", "a": "Yes — Enterprise tier includes a self-hosted option with full support." },
    { "q": "What about compliance?", "a": "SOC2 Type II certified. GDPR compliant. Data residency options available." }
  ]
}
```

## The Engine

### Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                     BROWSER (CLIENT)                          │
│                                                               │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────────────┐ │
│  │  SIGNAL      │  │  PERSONA     │  │  PAGE RENDERER      │ │
│  │  COLLECTOR   │──►  ENGINE      │──►                     │ │
│  │              │  │              │  │  Generates DOM from  │ │
│  │  scroll,     │  │  Infers who  │  │  block configs +    │ │
│  │  clicks,     │  │  they are &  │  │  product data       │ │
│  │  timing,     │  │  what they   │  │                     │ │
│  │  hover,      │  │  want        │  │  CSS-in-JS for      │ │
│  │  viewport    │  │              │  │  dynamic styling     │ │
│  └─────────────┘  └──────────────┘  └─────────────────────┘ │
│                          │                      │             │
│                    ┌─────▼──────┐         ┌─────▼──────┐     │
│                    │  MUTATION   │         │  ANALYTICS  │     │
│                    │  ENGINE     │         │  TRACKER    │     │
│                    │            │         │            │     │
│                    │  Real-time  │         │  Records   │     │
│                    │  page       │         │  what was  │     │
│                    │  adaptation │         │  shown &   │     │
│                    │  based on   │         │  how they  │     │
│                    │  behaviour  │         │  responded │     │
│                    └────────────┘         └────────────┘     │
└──────────────────────────────────────────────────────────────┘
```

### Generation Pipeline

**Phase 1: Initial Render (0ms — before paint)**
1. Read arrival signals (UTM, referrer, device, geo, time, return visit cookie)
2. Build initial persona estimate
3. Select block order, content variants, and visual properties
4. Render full page server-side or client-side

**Phase 2: Real-time Adaptation (ongoing)**
1. Signal collector sends behavioural events every 2 seconds
2. Persona engine updates confidence scores
3. Mutation engine applies non-jarring changes:
   - **Safe mutations** (user won't notice): swap upcoming sections they haven't scrolled to yet, adjust CTA text below the fold, load different testimonials
   - **Subtle mutations**: gradually adjust colour temperature, shift whitespace
   - **Never mutate**: content the user is currently reading, anything above current scroll position, form data

**Phase 3: Return Visit (next session)**
1. Read persona from cookie/localStorage
2. Skip awareness content, go straight to consideration/decision
3. Show new content they haven't seen
4. If they viewed pricing → lead with pricing comparison or ROI calculator
5. Reference their previous visit: "Welcome back — ready to start your trial?"

### The Generator (Not a Selector)

This is the critical distinction. Shapeshifter doesn't choose between pre-written pages. It **generates** everything at render time from raw product facts.

Given:
```json
{ "headline": "Deploy in minutes, not meetings" }
```

The generator **rewrites** this based on the persona:

| Persona | Generated headline |
|---------|-------------------|
| CTO / technical | "git push → production in 90 seconds. Zero YAML." |
| CEO / executive | "Your engineers spend 40% of their time on DevOps. We fix that." |
| Marketing manager | "Launch campaigns without waiting for a deploy queue" |
| Return visitor (saw pricing) | "Ready to start? Your free trial is waiting." |
| Developer from HN | "We replaced our entire CI/CD pipeline with one command" |

None of those headlines exist in the product JSON. They're **generated** from the raw facts (`headline`, `features[].metric`, `trust.deploys_per_day`) combined with the persona.

### Content Transforms

The generator applies transforms to raw content:

```typescript
interface Transform {
  // What to transform
  target: 'headline' | 'subheadline' | 'feature_description' | 'cta_text' | 'section_intro';
  
  // How to transform it
  operations: TransformOp[];
}

type TransformOp =
  | { op: 'reframe'; lens: 'cost' | 'speed' | 'risk' | 'control' | 'simplicity' | 'scale' }
  | { op: 'tone'; style: 'technical' | 'business' | 'casual' | 'urgent' | 'empathetic' }
  | { op: 'verbosity'; level: 'telegraphic' | 'concise' | 'standard' | 'detailed' }
  | { op: 'inject_metric'; from: string }  // pull a number from product data and weave it in
  | { op: 'personalise'; context: string }  // "for fintech teams" / "for your team"
  | { op: 'question'; flip: boolean }       // turn a statement into a question
  | { op: 'social_proof'; inline: boolean } // weave a testimonial quote into the copy
```

**Example: Feature description for "Auto Deploy"**

Raw fact:
> "Automatic builds triggered by git push. Supports Docker, Node, Python, Go, Rust. Blue-green deployments with automatic rollback on health check failure."

**Technical persona** (reframe: control, tone: technical, verbosity: detailed):
> "Push to main and it's live in 90 seconds. Builds run in isolated containers — Docker, Node, Python, Go, Rust, whatever your stack. Blue-green deployment with automatic rollback on health check failure. No Jenkins. No YAML. No pipeline debugging at midnight."

**Executive persona** (reframe: cost, tone: business, verbosity: concise, inject_metric):
> "Every code change goes live automatically, safely. Your team stops burning hours on deployment and starts shipping what matters. Teams using this deploy 30x faster."

**Casual/end-user** (reframe: simplicity, tone: casual, verbosity: telegraphic):
> "Push your code. It's live. If something breaks, it rolls back. That's it."

### Demo Transform Engine

In the demo, transforms are template functions — no LLM needed, but they produce genuinely different copy from the same source:

```typescript
// The transform engine takes raw content + persona and returns generated copy
function generateHeadline(product: Product, persona: Persona): string {
  const raw = product.headline;
  const metrics = product.features.map(f => f.metric).filter(Boolean);
  const topMetric = metrics[0];
  
  // Reframe based on what this persona cares about
  if (persona.role === 'technical-evaluator') {
    // Lead with the how — technical people want mechanism
    return extractMechanism(raw) + '. ' + topMetric + '.';
  }
  
  if (persona.role === 'executive-buyer') {
    // Lead with the problem — executives want the pain point
    const pain = inferPain(product);  // "your team wastes X hours on Y"
    return pain;
  }
  
  if (persona.intent === 'returning' && persona.previousPages?.includes('pricing')) {
    // They've done the research. Don't re-pitch. Nudge.
    return `Welcome back. Ready to ${product.pricing.tiers[1]?.name || 'get started'}?`;
  }
  
  if (persona.urgency === 'high') {
    // Strip everything. Just the promise and proof.
    return `${topMetric}. Try it free.`;
  }
  
  // Default: use the raw headline but inject the strongest metric
  return `${raw} — ${topMetric}`;
}

// Transform a feature description
function generateFeatureDescription(
  feature: Feature, 
  persona: Persona
): string {
  const raw = feature.description;
  const sentences = splitSentences(raw);
  
  // Technical: keep all detail, add specifics
  if (persona.sophistication === 'expert') {
    return raw + inferTechnicalDetails(feature);
  }
  
  // Business: extract outcome, drop mechanism
  if (persona.role === 'executive-buyer') {
    const outcome = extractOutcome(sentences);
    const metric = feature.metric;
    return metric ? `${outcome} ${metric}.` : outcome;
  }
  
  // Casual: first sentence only, simplified
  if (persona.content_preference === 'concise') {
    return simplify(sentences[0]);
  }
  
  return raw;
}

// Generate a CTA — never pre-written, always contextual
function generateCTA(product: Product, persona: Persona, position: 'hero' | 'mid' | 'footer'): { text: string; action: string; style: string } {
  
  if (persona.stage === 'decision') {
    return { text: 'Start your free trial', action: 'trial', style: 'strong' };
  }
  
  if (persona.trust_level === 'skeptical') {
    return { text: 'See how it works →', action: 'demo', style: 'soft' };
  }
  
  if (persona.role === 'executive-buyer') {
    return { text: 'Talk to our team', action: 'contact', style: 'medium' };
  }
  
  if (position === 'hero') {
    return { text: `Try ${product.name} free`, action: 'trial', style: 'medium' };
  }
  
  // Deeper in the page = warmer intent
  if (position === 'footer') {
    return { text: 'Get started — it\'s free', action: 'trial', style: 'strong' };
  }
  
  return { text: 'Learn more', action: 'learn', style: 'soft' };
}
```

### NLP Utilities (Rule-based, No LLM)

The demo includes simple NLP functions that enable genuine generation:

```typescript
// Extract the mechanism from a headline ("Deploy in minutes" → "git push → production")
function extractMechanism(text: string): string

// Infer the pain point from product data ("Your team spends 40% of time on DevOps")
function inferPain(product: Product): string

// Extract outcome sentences from a description
function extractOutcome(sentences: string[]): string

// Simplify a sentence (remove clauses, shorten)
function simplify(sentence: string): string

// Split a description into facts, mechanisms, and outcomes
function classifySentences(text: string): { facts: string[]; mechanisms: string[]; outcomes: string[] }

// Reframe a fact through a lens
function reframe(fact: string, lens: 'cost' | 'speed' | 'risk' | 'control'): string

// Generate a transition between two sections based on what the visitor just read
function generateTransition(fromBlock: BlockType, toBlock: BlockType, persona: Persona): string
```

### Behavioural Mutations (Live Rewriting)

As the visitor interacts, the page **rewrites sections they haven't reached yet**:

```typescript
interface Mutation {
  trigger: (signals: LiveSignals) => boolean;
  apply: (page: LivePage, product: Product, persona: Persona) => void;
  once: boolean;  // fire once or continuously
}

const mutations: Mutation[] = [
  {
    // Fast scroller losing interest → inject a pattern interrupt
    trigger: (s) => s.scrollVelocity > 800 && s.avgDwellPerSection < 3,
    apply: (page, product, persona) => {
      const nextSection = page.nextUnseenSection();
      if (!nextSection) return;
      // Generate a bold stat block from product data
      const stat = pickStrongestMetric(product, persona);
      page.injectBefore(nextSection, generateStatInterrupt(stat, persona));
    },
    once: false
  },
  {
    // Deep reader → upgrade upcoming sections to detailed mode
    trigger: (s) => s.scrollVelocity < 150 && s.totalDwellTime > 30,
    apply: (page, product, persona) => {
      persona.sophistication = 'expert';
      persona.content_preference = 'data-driven';
      page.upgradeUnseenSections(section => {
        section.regenerate(product, persona);  // re-run the generator with updated persona
      });
    },
    once: true
  },
  {
    // Hovering over CTA but not clicking → add reassurance
    trigger: (s) => s.ctaHoverTime > 2000 && !s.ctaClicked,
    apply: (page, product) => {
      const cta = page.nearestCTA();
      // Generate a trust line from product data
      const trustLine = `${product.trust.customer_count.toLocaleString()}+ teams trust us. ${product.trust.uptime} uptime. ${product.trust.certifications.join(', ')} certified.`;
      cta.addReassurance(trustLine);
      cta.softenText('See it in action first →');
    },
    once: true
  },
  {
    // Copying text → they're building a case for someone
    trigger: (s) => s.copyEvents > 0,
    apply: (page, product, persona) => {
      persona.intent = 'comparing-options';
      const nextSection = page.nextUnseenSection();
      if (nextSection) {
        page.injectBefore(nextSection, generateShareableBlock(product));
      }
    },
    once: true
  },
  {
    // Scrolled back up → re-reading, probably confused or comparing
    trigger: (s) => s.scrollReversals > 2,
    apply: (page, product, persona) => {
      // Regenerate the summary/CTA at the bottom with clearer language
      const footer = page.getSection('cta_strip');
      if (footer) {
        footer.setContent(generateClarityBlock(product, persona));
      }
    },
    once: true
  },
  {
    // Rage clicks → frustrated
    trigger: (s) => s.rageClicks > 0,
    apply: (page) => {
      page.injectFloating({
        type: 'help-prompt',
        content: 'Something not working? We\'re here →',
        position: 'bottom-right',
        action: 'chat'
      });
    },
    once: true
  }
];
```
```

### LLM Mode (Production)

In production, the transform engine is replaced by an LLM that does what the template functions do — but better:

```
System: You are generating a landing page for a specific visitor.

Product facts (raw, unformatted):
{product_json}

Visitor persona:
{persona_json}

Generate a complete page as a BlockConfig[] array. For each block:
- Write all copy from scratch. Do not use the raw product descriptions verbatim.
- Rewrite every headline, description, and CTA for this specific persona.
- Choose which product facts to emphasise and which to omit.
- Set layout, tone, verbosity, colour temperature, and CTA strength.
- Order the blocks for maximum persuasion given this visitor's intent and stage.

The visitor should feel like this page was written for them personally.
```

**Architecture for speed:**
1. **Edge function** (Cloudflare Workers / Vercel Edge) reads signals, builds persona
2. **LLM call** (~300ms with streaming) generates the page config
3. **Client renderer** streams blocks into the DOM as they arrive
4. **Behavioural mutations** remain rule-based (instant, no LLM latency)
5. **Cache by persona hash** — similar personas get the same cached generation, busted when product data changes

**Hybrid approach:** Use the LLM for the initial generation (headlines, descriptions, CTAs — the creative work) and rules for real-time mutations (injecting trust signals, softening CTAs, adding pattern interrupts). The LLM is slow but smart. The rules are fast but dumb. Use each where it fits.

## Demo Implementation

### Tech Stack
- **Vanilla TypeScript** — no framework dependency (it IS the framework)
- **Single HTML file** — the demo is self-contained
- **CSS custom properties** — dynamic theming via JS
- **Intersection Observer** — section visibility tracking
- **localStorage** — return visit memory
- **Navigator API** — device, connection, language

### Demo Product: "Versantus AI"
Use Versantus AI as the demo product — we have real content, real testimonials, real services. The demo shows the same product rendered completely differently for different visitors.

### Demo Modes (URL params)

Since the demo has no real referrer/UTM data, we simulate signals:

```
?persona=cto          → Simulates: GitHub referrer, desktop, fast connection
?persona=ceo          → Simulates: LinkedIn referrer, desktop, morning
?persona=marketer     → Simulates: Google Ads click, utm_campaign=growth-tools
?persona=returning    → Simulates: 3rd visit, previously viewed pricing + features
?persona=ppc          → Simulates: utm_medium=cpc, utm_campaign=deploy-faster
?persona=hackernews   → Simulates: HN referrer, fast scroll pattern
?persona=mobile       → Simulates: iPhone, evening, 4G connection, commute hours
?debug=true           → Show the debug panel (combinable with any persona)
```

Each mode doesn't load a different page — it sets the initial signals, and the generator runs the same pipeline it would for a real visitor. The page is generated fresh every time.

### Debug Panel

When `?debug=true`, a floating panel shows the generation happening in real-time:

**Left column — Inputs:**
- Raw signals (referrer, device, time, UTM, return visit data)
- Derived persona scores with confidence levels
- Behavioural signals updating live (scroll velocity, dwell time, etc.)

**Right column — Generation output:**
- Which transforms ran and what they produced
- Block order and why (e.g. "pricing moved to position 3 because persona.stage=decision")
- Generated copy diff — shows the raw product text → generated text side by side
- Active mutations log ("Section 'features' regenerated: verbosity upgraded to 'detailed'")

**Bottom strip — Simulate:**
- Buttons to trigger events: "Rage click", "Copy text", "Go idle", "Hover CTA", "Scroll fast"
- Each button fires the signal and you watch the page rewrite in real-time

### The "Shapeshifter Moment"

The demo's showpiece: split-screen mode.

```
?split=cto,ceo,marketer
```

Three columns. Same product JSON. Three completely different pages generated side by side. Different headlines. Different feature descriptions. Different block ordering. Different CTAs. Different colour temperature.

The visitor can watch the generator work — each column shows its persona badge, and you can see *why* each page looks the way it does. Hover over any piece of copy and it shows: "Generated from: {raw fact} → Transform: reframe(cost) + tone(business) + inject_metric"

This is the moment people get it. Not "we swapped a headline." The entire page is different. From the same data.

## File Structure

```
shapeshifter/
├── index.html              # Self-contained demo
├── product.json            # Versantus AI product data
├── src/
│   ├── engine/
│   │   ├── signals.ts      # Signal collector
│   │   ├── persona.ts      # Persona inference engine
│   │   ├── rules.ts        # Rule engine (demo mode)
│   │   ├── mutations.ts    # Real-time page mutation
│   │   └── analytics.ts    # What was shown, how they responded
│   ├── renderer/
│   │   ├── page.ts         # Page generator (blocks → DOM)
│   │   ├── blocks/         # One file per block type
│   │   │   ├── hero.ts
│   │   │   ├── features.ts
│   │   │   ├── social-proof.ts
│   │   │   ├── pricing.ts
│   │   │   ├── case-study.ts
│   │   │   ├── comparison.ts
│   │   │   ├── faq.ts
│   │   │   ├── cta-strip.ts
│   │   │   ├── trust.ts
│   │   │   ├── roi.ts
│   │   │   ├── technical.ts
│   │   │   ├── video.ts
│   │   │   └── urgency.ts
│   │   ├── themes.ts       # Dynamic colour/typography
│   │   └── animations.ts   # Entrance animations, transitions
│   ├── debug/
│   │   ├── panel.ts        # Debug overlay
│   │   └── split-view.ts   # Multi-persona comparison mode
│   └── types.ts            # All TypeScript interfaces
├── server.ts               # Simple Node HTTP server
└── README.md
```

## Why This Matters

This isn't just personalisation. Personalisation is "Hi {first_name}" and showing a different hero image. This is **generation** — the page doesn't exist until someone arrives, and it's never the same twice.

**Today**: Build 1 landing page. Hope it works for everyone. A/B test two headlines. Wait 3 months. Pick the winner. Repeat.

**With Shapeshifter**: Feed it your product facts. Every visitor gets a page that was *written* for them — not assembled from variants, not selected from templates. Generated. The copy is different. The arguments are different. The structure is different. The visual weight is different. The CTA is different.

**What this kills:**
- The "one page fits all" compromise
- Manually building /for-developers, /for-enterprise, /for-startups
- A/B testing as a strategy (when every visitor gets a unique page, the page *is* the test)
- The gap between what the ad promised and what the landing page delivers
- Bounce rates from irrelevant content (the page rewrites itself if you're losing interest)

**What this enables:**
- One URL, infinite experiences
- The page gets smarter during the visit (not between visits)
- Return visitors never see the same pitch twice
- Product data is the single source of truth — update the JSON, every generated page reflects it instantly
- Conversion data feeds back into generation rules — the system learns what works for each persona type

**The progression:**
1. **Demo** (this build): Template transforms. No LLM. Genuinely different pages from the same data, but the transforms are hand-coded functions.
2. **Production v1**: LLM generates copy at the edge. 300ms to a unique page. Cached by persona hash.
3. **Production v2**: The system trains on its own conversion data. Which generated pages convert? For which personas? The transforms evolve.
4. **Endgame**: You give it your product database and it generates your entire marketing site. Every page, every piece of copy, every CTA — all generated, all adaptive, all learning.

*The best landing page is the one that was written for exactly one person, three seconds ago.*
