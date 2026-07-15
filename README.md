# Product Research Bundle

A suite of **Claude skills** that take an Amazon-FBA product idea from *"which niche?"*
all the way to *"is it worth building, at what price, and what must the product fix?"* —
driven through your own logged-in Chrome, with real marketplace data and no fabrication.

Each skill is a **single `SKILL.md` file** you upload to a Claude agent. The agent drives
your browser (via the Claude-in-Chrome MCP), runs the analysis, and renders a static HTML
report. Nothing is invented — if a data source is blocked or missing, the skill stops and
asks you, rather than estimating.

---

## What's inside

| Skill | Role | Input | Output |
|---|---|---|---|
| [**product-opportunity-finder**](https://github.com/AronLEEdev/product-opportunity-finder-skill) | **Discovery** — turn a category into a ranked Top-3 niche shortlist | a category (e.g. *Kitchen & Dining*) | `opportunities.js` + shortlist report |
| [**product-cost-analyze-for-fba**](https://github.com/AronLEEdev/product-cost-analyze-skill) | **Cost** — is it profitable, and at what price? | a product keyword | `results.js` + margin report |
| [**product-review-analyze-skill**](https://github.com/AronLEEdev/product-review-analyze-skill) | **Review** — buyer pain points + market size + GO/WATCH/SKIP verdict | a product keyword | `reviews.js` + opportunity report, **+ combined dossier** |

The three run as a **pipeline**, and the review skill (the terminal step) also assembles a
single **Product Dossier** that stitches all three phases into one page.

```
   CATEGORY                    KEYWORD                        KEYWORD
      │                          │                               │
      ▼                          ▼                               ▼
┌───────────────┐         ┌──────────────┐              ┌──────────────────┐
│  DISCOVERY    │  pick a │    COST      │   validate   │     REVIEW       │
│ Top-3 niches  │ ──────▶ │  margins &   │ ──────────▶  │ pain points +    │
│ (Black Box)   │  niche  │  price band  │   the pick   │ market + verdict │
└───────────────┘         └──────────────┘              └──────────────────┘
                                                                  │
                                                                  ▼
                                                        ┌──────────────────┐
                                                        │  PRODUCT DOSSIER │
                                                        │ BUILD/TEST/PASS  │
                                                        │ all 3 in one page│
                                                        └──────────────────┘
```

---

## Quick start

1. **Set up the prerequisites** (below) — the one-time part.
2. **Upload a skill's `SKILL.md`** to your Claude agent (that's the whole install — no repo
   download; helper code and templates are embedded or fetched on demand).
3. **Ask** e.g. *"run the cost skill on `stainless steel bento box`"*. The agent connects to
   your Chrome, does the work, and opens the report at `http://localhost:<port>`.

Run the skills **in order** for a full workup: Discovery → Cost → Review. Each also works
standalone.

---

## Install

Two ways — pick whichever fits your setup.

### A. Claude Code plugin (one command)

This repo is also a **Claude Code plugin marketplace**. Add it once, then install any of the
three skills:

```bash
/plugin marketplace add AronLEEdev/product-research-skills
/plugin install product-opportunity-finder@product-research-skills
/plugin install product-cost-analyze@product-research-skills
/plugin install product-review-analyze@product-research-skills
```

Each installs as a self-contained plugin (one skill each) and shows up as a `/`-invocable
skill. Update anytime with `/plugin marketplace update product-research-skills`.

### B. Upload a single `SKILL.md` (any Claude agent)

The skills are plain single-file skills, so they work anywhere Claude runs — claude.ai, the
Agent SDK, or Claude Code — with **no install at all**. Grab a `SKILL.md` from its repo and
upload/paste it into the agent:

- Discovery → [product-opportunity-finder-skill](https://github.com/AronLEEdev/product-opportunity-finder-skill)
- Cost → [product-cost-analyze-skill](https://github.com/AronLEEdev/product-cost-analyze-skill)
- Review → [product-review-analyze-skill](https://github.com/AronLEEdev/product-review-analyze-skill)

Helper code is embedded in the file and report templates are fetched on demand, so one file is
the whole skill.

> **Cross-agent note:** the plugin and Skill formats are Claude-native. Non-Claude agents can't
> install a plugin, but the underlying automation these skills lean on (Claude-in-Chrome) is
> Claude-specific anyway. For cross-ecosystem tool interop, the portable path is MCP — e.g. the
> cost skill's fee math is also available as the [`profitlee-mcp`](https://www.npmjs.com/package/profitlee-mcp) server.

---

## Prerequisites (one-time setup)

Everything runs through **one logged-in Chrome profile**, so sign into each site there first.

| Requirement | Needed by | Required? |
|---|---|---|
| **Chrome + the Claude-in-Chrome MCP extension**, connected | all three | ✅ Required |
| **Amazon account**, logged in | all (SERP + review pages) | ✅ Required |
| **Node.js ≥ 18** | cost & discovery (helper scripts) | ✅ Required |
| **Python 3** (or any static server) | serving the HTML reports | ✅ Required |
| **1688.com account**, logged in | cost (FOB sourcing) | ✅ Required for cost |
| **Helium 10** extension + account | discovery (Black Box) · review (Cerebro/Xray) | ⭐ Strongly recommended |
| **Profitlee** (`profitlee-mcp`, or the free public API) | cost (FBA fee math) | ✅ Required for cost |

No API keys to manage: Profitlee's calc engine is free/public, and Helium 10 / Amazon / 1688
are driven through your existing logged-in sessions.

---

## The skills in detail

### 1 · Discovery — `product-opportunity-finder`
Turns a **category** into a ranked **Top-3 niche shortlist** (+ runners-up), each scored 0–100
and labelled **Strong / Worth-testing / Pass**. Pulls the full Black Box result set, scores it
deterministically (`score.mjs`: demand 40% / rankability 40% / margin 20%), and hands off the
best niche's keyword + example ASIN to the next two skills.

- **Tip:** point it at a **subcategory** (e.g. *Kitchen & Dining*, not top-level *Home & Kitchen*)
  — top-level results skew to furniture and distort scoring.

### 2 · Cost — `product-cost-analyze-for-fba`
Given a keyword, captures the top ~10 Amazon listings ($10–$50 band), reverse-image-searches
**1688** for real wholesale FOB, and runs each through the **Profitlee** FBA-fee engine. Output
is a ranked profitability table (net margin after PPC/returns/storage) with survivors, eliminated
listings, and a recommended price band.

- The FBA math lives in Profitlee's tested engine, not in the skill. Pack-count aware (landed
  COGS = 1688 unit price × Amazon pack count).
- **Caveat it surfaces:** 1688 image-matching can be noisy — check each row's `matchedTitle`
  before trusting a very-high margin.

### 3 · Review — `product-review-analyze-skill`
Mines critical + positive Amazon reviews for the top listings, clusters them into complaint /
praise themes with real quotes, and — when Helium 10 is available — adds a **market layer**:
Cerebro keyword demand + Xray 30-day revenue/units, producing a **GO / WATCH / SKIP** verdict
and an opportunity brief (must-fix / must-keep / differentiation angle).

- **Terminal step → Product Dossier.** If a sibling cost run exists, it also builds a one-page
  dossier fusing all three phases into a single **BUILD / TEST / PASS** call, so you read one
  report instead of three.

---

## 💰 Cost & metering — what each run actually consumes

There are **no per-run cash fees** (Profitlee is free; Amazon/1688 accounts are free). The real
budget is **Helium 10 metered lookups** and **Claude tokens**.

| Skill | Helium 10 metered | 1688 | Profitlee | Typical time | Re-run (cached) |
|---|---|---|---|---|---|
| Discovery | **≤ 3** Black Box | — | — | 5–10 min | **0 lookups** |
| Cost | **0** | 1 batch (creates a sourcing task) | ~10 free calls | 8–15 min | **0** — skips 1688 |
| Review | **1** Cerebro + a few Xray "load sales" | — | — | 8–15 min | **0** — incl. the metered Cerebro |

**Helium 10 metering:** Cerebro / Magnet / Black Box lookups draw from your **monthly plan quota**
(e.g. ~100/mo on lower tiers). Reading an already-open **Xray** overlay is *not* a lookup;
per-product "Load 30-day Sales Data" draws from Xray's separate, more generous quota. **Hard cap
of ≤ 3 metered lookups per run** is built into every skill.

**Claude tokens (rough):** each skill's `SKILL.md` is ~6–8K tokens to load, and a full run is
browser-heavy (many evals/screenshots). Expect a first run to be the pricier part; the static
template + `.mjs`-offloaded math mean **reports cost ≈0 tokens to render**, and **caching makes
re-runs cheap**.

**Caching (14-day TTL):** every skill caches its expensive scraped artifacts under
`~/Documents/product-research/_cache/<skill>/`. Re-running the same keyword/category within 14
days reuses them — **~0 browser work and 0 metered lookups**. Delete a cache file to force a
fresh pull.

---

## Best practices

- **Warm up all logins first** (Amazon, 1688, Helium 10) in the one Chrome profile before starting.
- **Run in order** — Discovery hands the best niche's keyword + ASIN to Cost and Review.
- **Discovery: use a subcategory**, not a top-level category, to avoid skew.
- **Respect the ≤ 3 metered-lookup cap** — the skills track it; cached artifacts count as 0.
- **Re-runs are cheap** — the 14-day cache means iterating on the same product is near-free.
- **Pricing: don't undercut.** The cost report's margin floor is real — below it you're buying
  market share with your own money. Premium/differentiated positioning usually wins on margin.
- **When sampling suppliers, test the #1 complaint on the sample** (from the review report), in
  front of the vendor. If it fails, walk — that's the failure that will tank your rating too.
- **Trust but verify FOB** — check `matchedTitle`; representative/estimated FOBs are flagged.

---

## Data integrity

These skills **never fabricate** reviews, sales, revenue, keywords, ASINs, or prices. When a
source is blocked (CAPTCHA, login wall) or a match fails, the skill **hard-stops and asks you**
to resolve it, then continues — it does not fall back to guesses. Cached data is only written
after a phase completes cleanly.

---

## Repository structure

The three skills are independently versioned repos, included here as **git submodules** so this
bundle can pin known-good versions while each skill evolves on its own:

```
product-research-bundle/
├── README.md                          ← you are here
├── product-opportunity-finder-skill/  ← submodule (Discovery)
├── product-cost-analyze-skill/        ← submodule (Cost)
└── product-review-analyze-skill/      ← submodule (Review)
```

Add the submodules once:

```bash
git submodule add https://github.com/AronLEEdev/product-opportunity-finder-skill.git
git submodule add https://github.com/AronLEEdev/product-cost-analyze-skill.git
git submodule add https://github.com/AronLEEdev/product-review-analyze-skill.git
git commit -am "add three product-research skills as submodules"
```

Clone the bundle with everything:

```bash
git clone --recurse-submodules https://github.com/AronLEEdev/product-research-skills.git
```

---

## Adding a future skill

New skills should follow the same conventions so they slot into the pipeline and the dossier:

1. **Single `SKILL.md`** — embed any helper code (like `calc.mjs` / `score.mjs`) verbatim; fetch
   only the HTML report template at runtime. No repo download required to use it.
2. **Static report** — write a `window.__<NAME>_DATA__` JS data file; a bundled HTML template
   renders it. Never regenerate HTML (keeps token cost ≈0).
3. **14-day cache** for any expensive scrape, under `~/Documents/product-research/_cache/<skill>/`.
4. **Deterministic math in a tested `.mjs`** (Node built-ins only) — offload arithmetic from the
   LLM, and **filter `null` before `Number()`** (the `Number(null) === 0` trap).
5. **No fabrication** — hard-stop and ask rather than estimate.
6. Add it here as a submodule and, if it produces a data file, wire it into the dossier.

---

*Built for Amazon-FBA sellers who want fast, honest, data-grounded product research — not vibes.*
