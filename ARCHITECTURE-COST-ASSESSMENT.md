# Architecture & Cost Assessment — Scaling Aieutics Diagnostics

**Date:** 2026-02-22
**Scope:** Current 2 Vercel projects → 5+ diagnostics with monetisation

---

## 1. Current State Audit

### 1.1 Repositories (7 total)

| Repo | Type | Hosting | Domain | Stack |
|------|------|---------|--------|-------|
| `aieutics-com.github.io` | Landing page | GitHub Pages | `resources.aieutics.com` | Static HTML |
| `poc-diagnostic` | Diagnostic | **Vercel** | `diagnostic.aieutics.com` | Next.js 16 / React 19 / Tailwind 4 |
| *(corporate variant)* | Diagnostic | **Vercel** | `corporatepoc.aieutics.com` | Next.js (assumed same stack) |
| `icp-clarity-diagnostic` | Diagnostic | GitHub Pages | `resources.aieutics.com/icp-clarity-diagnostic/` | Next.js 16 / React 19 / Tailwind 4 |
| `value-proposition-articulator` | Diagnostic | GitHub Pages | `resources.aieutics.com/value-proposition-articulator/` | Next.js 16 / React 19 / Tailwind 4 |
| `pricing-coherence-diagnostic` | Diagnostic | GitHub Pages | `resources.aieutics.com/pricing-coherence-diagnostic/` | Next.js 16 / React 19 / Tailwind 4 |
| `critical-path-startups` | Framework | GitHub Pages | Subdirectory | Static HTML |
| `critical-path-corporate` | Framework | GitHub Pages | Subdirectory | Static HTML |

### 1.2 Dependency Matrix — Diagnostics Only

| Capability | poc-diagnostic | icp-clarity | value-prop | pricing-coherence |
|-----------|:-:|:-:|:-:|:-:|
| Next.js 16 + React 19 | Yes | Yes | Yes | Yes |
| Tailwind CSS v4 | Yes | Yes | Yes | Yes |
| Recharts (charts) | Yes | Yes | Yes | Yes |
| Vercel Analytics | Yes | Yes | Yes | Yes |
| Vercel Speed Insights | Yes | — | Yes | Yes |
| **Resend** (email) | Yes | — | — | — |
| **jsPDF** (PDF reports) | Yes | — | — | — |
| **jose** (JWT auth) | Yes | — | — | — |
| **Notion API** (data store) | Yes | — | — | — |

**Key finding:** `poc-diagnostic` is the only feature-complete diagnostic. The other 3 are front-end-only with no email, PDF, auth, or data storage. They currently run as static exports on GitHub Pages. To match the POC diagnostic's capabilities (and to monetise), each will need those same 4 services added.

### 1.3 Current Monthly Cost

| Item | Cost |
|------|------|
| Vercel Pro (1 seat) | $20 |
| Resend (free tier, <3K emails/month) | $0 |
| Notion API (free) | $0 |
| Domain (`aieutics.com` renewal, amortised) | ~$1 |
| GitHub Pages | $0 |
| **Total** | **~$21/month** |

---

## 2. Target Architecture — 5 Diagnostics at Parity

### 2.1 The 5 diagnostics to bring to production quality

1. **POC Lifecycle Diagnostic** — live on Vercel ✓
2. **Corporate Innovation Diagnostic** — live on Vercel ✓
3. **ICP Clarity Diagnostic** — needs upgrade
4. **Value Proposition Articulator** — needs upgrade
5. **Pricing Coherence Diagnostic** — needs upgrade

Each needs: scoring + charts + email report + PDF export + data persistence + auth/gating.

### 2.2 Architecture Options

#### Option A: Multi-repo / Multi-project (current path)

```
5 repos → 5 Vercel projects → 5 subdomains
```

| Pros | Cons |
|------|------|
| Full isolation per diagnostic | Massive code duplication |
| Independent deploy cycles | 5x the maintenance surface |
| Easy to reason about individually | Every shared fix must be applied 5 times |
| Current approach — no migration needed | Adding diagnostic #6 means cloning a repo |

**Verdict:** Works today but does not scale. By diagnostic #5 the maintenance overhead becomes the bottleneck.

#### Option B: Turborepo monorepo with shared packages

```
1 repo / packages/ui + packages/scoring → 5 Vercel projects (apps/)
```

| Pros | Cons |
|------|------|
| Shared components, single source of truth | Higher initial setup complexity |
| Independent deployment per diagnostic | Vercel monorepo config required |
| Adding diagnostic = new `apps/` folder | Build times grow with repo size |
| Can migrate incrementally from current state | |

**Verdict:** Good engineering choice for 5-8 diagnostics. Balances code sharing with deployment isolation.

#### Option C: Single Next.js app, config-driven (recommended for cost)

```
1 repo → 1 Vercel project → route-based diagnostics
  /d/poc-lifecycle
  /d/icp-clarity
  /d/value-proposition
  /d/pricing-coherence
  /d/corporate-innovation
```

Each diagnostic is defined by a **configuration file** (questions, dimensions, scoring weights, copy, branding). The app renders any diagnostic from config.

| Pros | Cons |
|------|------|
| 1 project, 1 deployment, 1 codebase | Single point of failure |
| Adding diagnostic = adding a TS config file | Shared deploy — one bad merge affects all |
| Shared infra (Resend, Notion, analytics) | URL structure changes from current subdomains |
| Lowest possible hosting cost | Requires upfront refactor |
| Easiest to add gating/auth/payments once | |

**Verdict:** Best unit economics and fastest path to adding diagnostics 6, 7, 8+. Can still use subdomain routing via Next.js middleware if branded URLs matter.

### 2.3 Recommendation

**Start with Option C** (single app) for the 3 beta diagnostics. Keep the 2 existing Vercel projects live during migration. Once validated, fold the existing 2 into the unified app.

This means you operate **3 Vercel projects temporarily** (existing 2 + new unified), then consolidate to **1 Vercel project** for all diagnostics.

---

## 3. Cost Scenarios

### 3.1 Platform costs — Vercel Pro

Vercel Pro is **$20/month per developer seat**, not per project. Whether you run 1 project or 10, the platform fee is the same $20/month (single-developer team).

| Scenario | Vercel Projects | Platform Cost |
|----------|:-:|:-:|
| Current (2 projects) | 2 | $20/month |
| Multi-repo (5 projects) | 5 | $20/month |
| Single app (1 project) | 1 | $20/month |

**Vercel Pro includes per month:**
- 1 TB bandwidth (~enough for 500K+ diagnostic completions)
- 1,000 GB-hours serverless compute
- $20 usage credit (offsets overages)
- Unlimited projects

### 3.2 Service costs at scale

| Service | Free Tier | First Paid Tier | When You'll Need Paid |
|---------|-----------|----------------|----------------------|
| **Resend** (email) | 3,000 emails/month | $20/month (50K emails) | >3,000 completions with email/month |
| **Notion API** | Free (API calls unlimited) | — | Never (for this use case) |
| **Vercel Analytics** | Included in Pro | — | Already covered |
| **Custom domains** | Subdomains of aieutics.com | — | $0 additional |
| **Stripe** (if monetised) | 2.9% + 30¢ per transaction | — | From first paid transaction |

### 3.3 Total monthly cost by usage volume

Assumes 5 diagnostics live, single-app architecture, email sent per completion, optional PDF.

| Monthly Completions | Vercel | Resend | Stripe Fees | **Total Fixed** | **Total w/ Stripe** |
|:---:|:---:|:---:|:---:|:---:|:---:|
| 100 | $20 | $0 | — | **$20** | $20 |
| 500 | $20 | $0 | — | **$20** | $20 |
| 1,000 | $20 | $0 | — | **$20** | $20 |
| 3,000 | $20 | $0 | — | **$20** | $20 |
| 5,000 | $20 | $20 | — | **$40** | $40 |
| 10,000 | $20 | $20 | — | **$40** | $40 |
| 50,000 | $20 | $90 | — | **$110** | $110 |

**Key insight:** Costs are effectively flat at $20/month until you exceed 3,000 email sends/month. You can run 5 diagnostics for the same $20 you pay for 2 today.

### 3.4 Per-project cost allocation

With the single-app architecture, the per-project cost is simply:

| Diagnostics Live | Cost per Diagnostic/month |
|:---:|:---:|
| 2 (current) | $10.00 |
| 5 | $4.00 |
| 7 | $2.86 |
| 10 | $2.00 |

Every diagnostic you add **lowers the per-unit cost** since the platform fee is fixed.

### 3.5 Alternative: Cloudflare Pages (cost comparison)

| | Vercel Pro | Cloudflare Pages (Free) | Cloudflare + Workers ($5) |
|---|:---:|:---:|:---:|
| Platform fee | $20/month | $0 | $5/month |
| Commercial use | Yes | Yes | Yes |
| Bandwidth | 1 TB | **Unlimited** | **Unlimited** |
| Serverless | 1,000 GB-hrs | 100K requests/day free | 10M requests/month |
| Next.js support | Native (Vercel built it) | Via `@cloudflare/next-on-pages` | Same |
| Deploy preview URLs | Yes | Yes | Yes |
| Analytics | Built-in | Cloudflare Web Analytics (free) | Same |

**Cloudflare trade-off:** Saves $15-20/month but Next.js support is adapter-based rather than native, and some Next.js features (ISR, middleware, image optimisation) work differently or not at all. If the diagnostics are primarily client-rendered with simple API routes, Cloudflare is viable. If you rely on Vercel-specific features, stay on Vercel.

**Verdict:** The $20/month difference is immaterial relative to the monetisation revenue. Stay on Vercel for native Next.js support unless cost pressure becomes real.

---

## 4. Monetisation Viability

### 4.1 Revenue model assumptions

| Tier | What user gets | Price point |
|------|---------------|------------|
| **Free** | Complete the diagnostic, see summary score | $0 (lead generation) |
| **Report** | PDF with full breakdown, dimension analysis, recommendations | $10–25 per report |
| **Advisory** | Diagnostic results + 30-min advisory call | $150–350 per session |

### 4.2 Unit economics per diagnostic completion

| | Free tier user | Paid report user |
|---|:---:|:---:|
| Hosting cost | ~$0.004 | ~$0.004 |
| Email cost | $0 (within free tier) | ~$0.0004 |
| PDF generation | $0 (client-side) | $0 (client-side) |
| Stripe fee | — | $0.59–$1.03 (2.9%+30¢) |
| **Total COGS** | **~$0.004** | **~$0.60–$1.03** |
| **Revenue** | $0 | $10–25 |
| **Gross margin** | — | **94–96%** |

### 4.3 Breakeven analysis

Monthly fixed costs: ~$20 (Vercel Pro)

| Price per report | Reports to breakeven | Completions at 5% conversion |
|:---:|:---:|:---:|
| $10 | 3 reports/month | 60 completions |
| $15 | 2 reports/month | 40 completions |
| $25 | 1 report/month | 20 completions |

**The monetisation model is sound.** Fixed costs are so low that breakeven occurs with 1–3 paid reports per month across all 5 diagnostics. The marginal cost per additional diagnostic or completion is effectively zero until you hit 3,000 emails/month.

### 4.4 Revenue scenarios (5 diagnostics live)

| Monthly Traffic | Conversion Rate | Avg. Price | **Monthly Revenue** | **Monthly Cost** | **Margin** |
|:---:|:---:|:---:|:---:|:---:|:---:|
| 500 completions | 3% | $15 | $225 | $20 | $205 |
| 1,000 completions | 5% | $15 | $750 | $20 | $730 |
| 3,000 completions | 5% | $15 | $2,250 | $20 | $2,230 |
| 5,000 completions | 5% | $15 | $3,750 | $40 | $3,710 |
| 10,000 completions | 5% | $15 | $7,500 | $40 | $7,460 |

---

## 5. Implementation Roadmap

### Phase 1 — Shared diagnostic engine (single app)

- Create unified Next.js app with config-driven diagnostic renderer
- Define schema: questions, dimensions, scoring weights, copy, branding per diagnostic
- Port the 3 beta diagnostics (ICP, Value Prop, Pricing) into the new app
- Deploy on Vercel as a single project (e.g., `diagnostics.aieutics.com`)

### Phase 2 — Feature parity

- Add Resend email integration (shared across all diagnostics)
- Add jsPDF report generation (shared template, per-diagnostic config)
- Add Notion API integration for storing completions
- Add JWT-based gating (jose) for premium reports

### Phase 3 — Monetisation layer

- Integrate Stripe Checkout for paid PDF reports
- Implement free tier (score only) vs paid tier (full report + email)
- Add conversion tracking (completion → email capture → paid report)

### Phase 4 — Consolidate existing diagnostics

- Migrate POC Lifecycle and Corporate Innovation into the unified app
- Retire the 2 standalone Vercel projects
- Redirect old subdomains to new routes

---

## 6. Decision Summary

| Decision | Recommendation | Rationale |
|----------|---------------|-----------|
| **Architecture** | Single Next.js app, config-driven | Lowest cost, least maintenance, fastest to add new diagnostics |
| **Hosting** | Vercel Pro | Native Next.js, already in use, $20/month flat covers everything |
| **Email** | Resend (free tier → Pro) | Already integrated in POC diagnostic, 3K/month free |
| **Data store** | Notion API (free) | Already integrated, sufficient for diagnostic submissions |
| **Payments** | Stripe | Industry standard, 2.9%+30¢, no monthly fee |
| **PDF generation** | jsPDF (client-side) | Zero server cost, already proven in POC diagnostic |
| **Auth/gating** | jose JWT | Lightweight, already in POC diagnostic, no external service needed |
| **Move off Vercel?** | No (not yet) | $20/month saving is immaterial vs monetisation revenue |

### Cost summary

| Scenario | Monthly Cost | Per Diagnostic |
|----------|:-:|:-:|
| **Today** (2 diagnostics) | $21 | $10.50 |
| **5 diagnostics** (<3K completions) | $21 | $4.20 |
| **5 diagnostics** (3K–5K completions) | $41 | $8.20 |
| **7 diagnostics** (<3K completions) | $21 | $3.00 |

---

*Sources: [Vercel Pricing](https://vercel.com/pricing) · [Cloudflare Pages Pricing](https://www.cloudflare.com/plans/developer-platform/) · [Resend Pricing](https://resend.com/pricing) · Dependency data from GitHub repos*
