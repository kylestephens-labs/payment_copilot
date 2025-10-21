🧠 Payment Copilot — MVP Spec (AWS Edition, 5-Day Launch)

Core Goal: Deliver financial clarity in 5 minutes.
Connect Stripe + PayPal → instantly see lifetime earnings, fees, and upcoming payouts.
No daily summary emails in MVP. No waitlist. Not Stripe-only.

A) Day-by-Day Plan (5 Days)

Day 1 — Landing Page + Arcade Demo + CTA
	•	Next.js static page with Tailwind.
	•	Hero: “Your creator money, finally in one place.”
	•	CTA: “Start Free 14-Day Trial” → email entry (no password).
	•	Arcade demo embed below CTA (90s–2m guided walkthrough).
	•	Sections: Hero → Demo → Feature grid (Timeline, Fees, Forecast, Export) → (future) testimonials → Footer (Terms/Privacy/Support).
	•	Email sending via SES or Postmark for magic links.

Day 2 — Email Auth (Gmail + others) + Account & Trial
	•	Flow: enter email → create user + trial_expires_at = now +14d → send signed magic link → /welcome?token=XYZ.
	•	Cognito Hosted UI (email magic link) or custom Lambda that issues short-lived JWT.
	•	Database tables:
	•	users(id, email, created_at, trial_expires_at, plan, status)
	•	sessions(id, user_id, issued_at, expires_at, jwt_id)
	•	Security: single-use tokens, 15-minute validity, rotate JWT secrets.

Day 3 — Stripe Integration (OAuth + 90-day Import + Webhooks)
	•	OAuth connect (standard Stripe Connect style for data access).
	•	Initial backfill (last 90 days): charges (gross, fee, net, created), payouts.
	•	Webhooks: charge.succeeded, payout.*, balance.available.
	•	Store to RDS using normalized schema (see “Data Model” below).
	•	UI: Dashboard shows “Importing Stripe…” counter and partial preview cards.

Day 4 — PayPal Integration (OAuth + 90-day Import)
	•	OAuth connect; fetch transactions + payouts for 90 days.
	•	Webhook if available, else 6h polling fallback.
	•	Same normalization as Stripe.
	•	UI: “Importing PayPal history…” progress bar + partial preview.

Day 5 — Dashboard MVP + Paywall + QA + Launch
	•	Dashboard Sections (must feel alive immediately):
	1.	Unified Timeline: Gross, fees, net by day/week; toggle platform.
	2.	Platform Breakdown: Stripe vs PayPal %, fees this month.
	3.	Fee Analyzer: Effective take rate + “You lost $___ to fees this month.”
	4.	Forecast Cards:
“Next payout: Stripe $X (Fri) • PayPal $Y (Tue)”
	5.	Lifetime Earnings: “You’ve earned $Z total (+MoM%)”
	6.	Tax Export: CSV + PDF → “Ready for accountant.”
	7.	Quick Actions: “Connect both → View fees → Export tax pack.”
	•	Paywall (Stripe Billing): in-app modal: “Continue full access — $39.99/mo” (auto after trial).
	•	Polish/QA: Empty states, error banners (“Waiting for data from Stripe”), reconciliation spot-checks.
	•	Launch: Push live, share the demo (Twitter/X, Product Hunt, Indie Hackers), and send targeted plain-text outreach to 50–100 creators.

B) Instant-Value Onboarding (Preview → Full Import)
	•	Immediately after connecting a platform, show preview metrics while the full import runs:
	•	“Stripe: $2,847 this month (2.9% fees)”
	•	“PayPal: $1,203 this month (2.5% fees)”
	•	“Total: $4,050 (+23% vs last month)”
	•	Micro-celebrations:
“🎉 Connected both platforms!” / “📈 You’re up 23% MoM!” / “💰 $1,000+ earned this month!”

C) Data Model (RDS Postgres)

-- providers: 'stripe' | 'paypal'
create table users (
  id uuid primary key,
  email text unique not null,
  created_at timestamptz not null default now(),
  trial_expires_at timestamptz,
  plan text default 'trial', -- trial|starter
  status text default 'active' -- active|past_due|canceled
);

create table connections (
  id uuid primary key,
  user_id uuid references users(id),
  provider text not null,
  access_token text not null, -- encrypted at rest
  account_id text not null,
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);

create table transactions (
  id uuid primary key,
  user_id uuid references users(id),
  provider text not null,
  ext_id text not null, -- charge id / transaction id
  gross_cents bigint not null,
  fee_cents bigint not null,
  net_cents bigint not null,
  currency char(3) not null,
  occurred_at timestamptz not null,
  created_at timestamptz default now(),
  unique(user_id, provider, ext_id)
);

create table payouts (
  id uuid primary key,
  user_id uuid references users(id),
  provider text not null,
  ext_id text not null, -- payout id
  amount_cents bigint not null,
  currency char(3) not null,
  status text not null, -- pending|paid|failed
  arrival_date date,
  created_at timestamptz default now(),
  unique(user_id, provider, ext_id)
);

create index idx_tx_user_date on transactions(user_id, occurred_at);
create index idx_payout_user_date on payouts(user_id, arrival_date);

D) API Surface (Lambda + API Gateway)
	•	POST /auth/magic-link → send magic link
	•	POST /auth/consume → exchange token for session
	•	POST /oauth/stripe/callback → store tokens, enqueue backfill
	•	POST /oauth/paypal/callback → store tokens, enqueue backfill
	•	POST /webhooks/stripe → upsert tx/payouts
	•	POST /webhooks/paypal (or polling job) → upsert tx/payouts
	•	GET /dashboard/summary → totals, fees, MoM
	•	GET /dashboard/timeline?interval=day|week
	•	GET /dashboard/forecasts → next payouts by provider
	•	POST /export/tax → produce CSV/PDF to S3 (presigned URL)

E) Forecast Logic (MVP)
	•	Stripe & PayPal payout schedule defaults (express/standard) + observed cadence from last 90 days.
	•	Estimate next arrival date from last paid payout + typical interval; display provider-specific cards.
	•	Show confidence badge (e.g., “High / Medium”) based on pattern detection.

F) Frontend (Next.js + Tailwind)
	•	Load ≤ 3s, optimistic UI for imports with animated counters.
	•	Plain-language tooltips (“Includes platform + processing fees”).
	•	Confetti/glow when both connectors complete first sync.
	•	Accessibility: keyboard navigable, reduced motion fallback.

G) Analytics & Metrics (PostHog/Plausible)
	•	Activation: % who connect Stripe; % who connect both; time-to-WOW.
	•	Feature usage: Fee Analyzer views, Export clicks, Forecast card interactions.
	•	Conversion: Trial → Paid (modal views, checkout start/complete).
	•	Quality: Import error rate; average time to first numbers; data confidence mix.

H) Security & Compliance (MVP)
	•	Encrypt tokens at rest (KMS), rotate secrets, least-privilege IAM.
	•	Signed webhooks with replay protection.
	•	GDPR basics: deletion endpoint, privacy policy, data retention schedule.
	•	No PII beyond email; no card data stored (only aggregated financial events).

I) “WOW” Checklist (Ship Gate)
	•	✅ Instant visual feedback on first connect.
	•	✅ Fee awareness: “You lost $___ to fees in the past 90 days.”
	•	✅ Forecast certainty: provider-specific payout cards with dates.
	•	✅ Pride moment: Lifetime earnings + MoM growth.
	•	✅ Tangible utility: “Download tax pack” working within minutes.

⸻