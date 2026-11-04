# KUWAPANI.COM — E-commerce starter package

This is a working starter storefront using Next.js + Supabase Postgres/Storage + Stripe Checkout + Vercel hosting. It is designed so you can configure the real accounts yourself. It is **not yet a fully audited production marketplace**.

## What is included
- Responsive storefront with product cards, cart and Stripe-hosted checkout.
- Server-side product/stock validation before creating checkout.
- Stripe webhook endpoint to record paid orders.
- Supabase SQL schema with public product read policy and private order table.
- Product photo storage bucket.
- Excel workbook for products/inventory, orders, expenses and launch checklist.
- `.env.example` with every manual field clearly marked `[ADD ...]`.

## Accounts you need
1. Domain registrar account for `kuwapani.com` (confirm domain ownership and DNS access).
2. GitHub account — stores source code. Do not upload `.env.local`, passwords, banking details or secret keys.
3. Supabase account — database, product image storage and backend keys.
4. Stripe account — payment processing, identity/business verification and bank payout setup.
5. Vercel account — deploys the Next.js site and connects GitHub/domain.

Free tiers and provider policies/pricing can change. Review current terms before relying on a free plan.

## Folder map
- `app/` — pages and API endpoints
- `components/` — storefront/cart interface
- `lib/` — database clients, product queries and money formatting
- `supabase/schema.sql` — run once in Supabase SQL Editor
- `.env.example` — copy to `.env.local`; replace highlighted placeholders
- `Kuwapani-Operations.xlsx` — editable product and business workbook

## Setup in order

### 1. Download and prepare
Extract the ZIP. Install Node.js LTS from https://nodejs.org and Git from https://git-scm.com.
Open a terminal in the extracted `kuwapani-ecommerce` folder:

```bash
npm install
cp .env.example .env.local
npm run dev
```
On Windows PowerShell, use `Copy-Item .env.example .env.local` instead of `cp`.
Open http://localhost:3000. Until credentials are set, sample products appear; checkout stays disabled with a clear configuration message.

### 2. Supabase database and storage
- Create a project at https://supabase.com/dashboard.
- Save the database password securely in a password manager.
- Open Project Settings / API (or Connect) and copy the Project URL and publishable/anon key.
- Open SQL Editor > New query; paste all of `supabase/schema.sql`; click Run.
- Open Storage and confirm `product-images` bucket exists.
- Upload product photos into that bucket. Use the public URL as `image_url` in the `products` table.
- Copy the **service_role / secret key** for server environment only. Never place it in a browser variable or GitHub.

### 3. Add Supabase values locally
Open `.env.local` and replace:
- `NEXT_PUBLIC_SUPABASE_URL` = Supabase Project URL
- `NEXT_PUBLIC_SUPABASE_ANON_KEY` = publishable/anon key
- `SUPABASE_SERVICE_ROLE_KEY` = service role/secret key (server only)

Do not send credentials to anyone or paste them in chat. Restart `npm run dev` after editing.

### 4. Stripe payments
- Create a Stripe account at https://dashboard.stripe.com/register.
- Complete Stripe's business/identity verification and enter the bank account where payouts should go.
- Start in Test mode. Copy the **test secret key** into `STRIPE_SECRET_KEY`.
- Install Stripe CLI from https://stripe.com/docs/stripe-cli and sign in.
- In a second terminal run:
```bash
stripe listen --forward-to localhost:3000/api/stripe-webhook
```
- Copy the webhook signing secret shown by the CLI into `STRIPE_WEBHOOK_SECRET`.
- Test a payment using Stripe's documented test card `4242 4242 4242 4242`, a future expiry, any CVC and postal code. Use test mode only.
- In production, create a webhook endpoint in Stripe Developers > Webhooks:
  `https://kuwapani.com/api/stripe-webhook`
  Subscribe to `checkout.session.completed`. Copy its `whsec_...` signing secret to the production environment.
- Replace the test Stripe secret with the live secret only after all tests, legal pages, products, shipping and tax settings are ready.

Stripe Checkout hosts the payment form; this project does not store card numbers or bank details. Stripe account approval and payout timing are controlled by Stripe.

### 5. GitHub — store your code
Create a **private** repository named `kuwapani-store` at https://github.com/new.
From this folder:
```bash
git init
git add .
git commit -m "Initial Kuwapani storefront"
git branch -M main
git remote add origin https://github.com/YOUR-GITHUB-USERNAME/kuwapani-store.git
git push -u origin main
```
Replace `YOUR-GITHUB-USERNAME`. `.gitignore` prevents `.env.local` from being committed. Before pushing, verify it is not staged:
```bash
git status
```
If a secret was ever committed, rotate it immediately in the provider dashboard; deleting the file from the latest commit is not enough.

### 6. Vercel hosting and domain
- Sign up at https://vercel.com.
- Add New Project > Import Git Repository > select `kuwapani-store`.
- Add the environment variables from `.env.local` in Vercel Project Settings > Environment Variables. Add them to Production (and Preview if needed). Never put secret keys in `NEXT_PUBLIC_` variables.
- Set `NEXT_PUBLIC_SITE_URL` to `https://kuwapani.com`.
- Deploy. Add `kuwapani.com` and `www.kuwapani.com` under Project Settings > Domains.
- At your domain registrar, copy the exact DNS records Vercel displays. DNS changes can take time.
- Redeploy after changing environment variables.
- Configure the Stripe production webhook URL after domain is active.

### 7. Add products
Use Supabase Dashboard > Table Editor > products > Insert row. Fill:
- `name`, `description`, `sku`
- `price_cents`: amount in US cents (e.g. $24.99 = 2499)
- `category`, `stock_quantity`, `image_url`, `is_active`
- `is_active=true` publishes the product.
The sample products can be edited or deleted. The Excel workbook is a working inventory template, not an automatic sync with Supabase.

## Before accepting real orders — mandatory checks
- Test successful, declined and cancelled checkout in Stripe Test mode.
- Confirm webhook records paid orders in Supabase.
- Test stock validation and verify no customer can read the orders table.
- Add real product images, accurate descriptions, prices and stock.
- Add and publish: Contact, About, Shipping, Returns/Refunds, Privacy, Terms, business identity and customer support details.
- Decide shipping zones, shipping charges, fulfillment process, sales tax responsibilities and return handling. Get professional tax/legal advice for your business situation.
- Confirm Stripe live-mode verification and bank payout details are approved.
- Confirm domain HTTPS, email address, order support and backups.
- Place a real low-value test order only after launch readiness and refund it if appropriate.

## Important limitations in this starter
- Retail store checkout only. It does not yet include customer accounts, password reset, admin dashboard, discount codes, automated shipping labels, sales-tax calculation, inventory decrement/reservation, refund dashboard, email notifications, analytics or seller marketplace payouts.
- Order records are created by the Stripe webhook. The current webhook records the paid checkout and customer details; it does not yet create detailed order-item rows or decrement stock. Add these before scaling.
- This is a foundation, not a guarantee of next-day commercial readiness. Have a developer review security, tax, privacy, accessibility, webhook retries/idempotency, inventory race conditions and fulfillment before taking significant orders.

## Provider references
- Supabase Next.js quickstart: https://supabase.com/docs/guides/getting-started/quickstarts/nextjs
- Supabase Auth/SSR: https://supabase.com/docs/guides/auth/server-side
- Stripe Checkout: https://docs.stripe.com/checkout/quickstart
- Stripe webhooks: https://docs.stripe.com/webhooks
- Stripe Connect (for future marketplace sellers): https://stripe.com/connect/marketplaces
- Vercel deployment: https://vercel.com/docs/deployments
- Vercel environment variables: https://vercel.com/docs/environment-variables
