# FCA — Fitness Content Agent

## What This Project Is
FCA is an AI-powered social media content platform for fitness studios. It generates captions, images, and video reels tailored to each studio's brand, instructors, and schedule.

## Pricing
- **Base plan:** $400/month
- **Instructor seats:** $60/month per instructor

## Launch Status
- Launched March 2026
- Active beta at **The Local Kollective** (instructors: Katie, Jackie, Ella)
- Core features live: AI content generation, photo editor, video reels, instructor seat management, Stripe billing

## Stack
| Layer | Service | Detail |
|---|---|---|
| Frontend | Netlify | `studio-dash.netlify.app` |
| Backend / API | Digital Ocean | Node.js server |
| Database | Supabase | Project ID: `kidgcrqxrfcbsaeguwop` |
| Payments | Stripe | Webhook workflow: `q7IuW7Q85mpgOYub` on n8n |
| Automation | n8n | `jmac.app.n8n.cloud` |

## Supabase Schema — Key Tables
- `studios` — one row per studio account
- `trials` — auto-created via DB trigger on studio insert
- `studio_instructors` — instructor seats per studio
- `leads` — outreach prospects (Lead Gen)
- `lc_leads` — leads from The Local Kollective channel
- `lc_lead_events` — activity events on lc_leads
- `clients` — active paying clients
- `content_deliveries` — generated content records
- `fitness_content` — content library
- `email_templates` — outreach email templates
- `commission_ledger` / `commission_payouts` — affiliate/sales rep tracking
- `referral_sources` / `search_keywords` — attribution data
- `trial_config` — global trial settings
- `agents` — AI agent configurations
- `reel_music_library` — music assets for reels

### Views (read-only, do not modify)
- `v_attribution_dashboard`
- `v_channel_performance`
- `v_hot_leads_today`
- `v_monthly_payout_report`
- `v_sales_rep_dashboard`

## Trial System
Every new studio gets a free trial automatically on signup. No credit card required.

**Trial limits:**
- 5 days access, OR
- 5 generated posts
Whichever is hit first ends the trial.

**Trial statuses:** `active`, `expired`, `converted`

**Beta accounts:** Any studio with `is_beta = true` on `studios` is permanently exempt from all trial logic. Never modify trial behavior for beta accounts.

**Attribution:** The `trials` table has `source` (text) and `lead_gen_id` (text) columns. When a studio signs up via an outreach link, these are populated from URL params:
```
https://studio-dash.netlify.app/signup?source=email_outreach_march2026&lead_gen_id=abc123
```
`lead_gen_id` maps to `leads.id` (uuid). Two DB triggers handle attribution back to the `leads` table:
- `on_trial_created` — sets `signup_completed_at` and `trial_started = true` on INSERT
- `on_trial_converted` — sets `converted = true` on UPDATE when `trial_status = converted`

**Conversion:** When a studio converts, the Stripe webhook fires → n8n workflow `q7IuW7Q85mpgOYub` → sets `trial_status = converted` and `converted_at = NOW()` on the `trials` row.

## Current Focus
Customer acquisition and lead generation. The trial system and attribution loop are the active build area.

## Important Rules
- **Never touch beta studio accounts** (`is_beta = true`)
- **Never modify existing triggers** without reading them first — there is already a trigger on `studios` INSERT that creates the `trials` row
- **Always check views before querying raw tables** for reporting — the views are pre-built for attribution and performance data
- **The Stripe webhook workflow ID is `q7IuW7Q85mpgOYub`** — reference by ID, do not recreate
- Signup URL is `https://studio-dash.netlify.app/signup`
