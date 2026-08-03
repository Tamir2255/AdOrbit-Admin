# AdOrbit / OrbitEarn — Full Platform

A twin-platform advertising network:
- **AdOrbit** (`adorbit-advertiser/`) — advertisers create and pay for campaigns.
- **OrbitEarn** (`orbitearn-earner/`) — earners complete tasks and get paid.
- **Admin Console** (`admin-console/`) — content generation queue, social proof review, and follower-tier verification.
- **Backend** (`backend/`) — one shared Node.js/Express API for all three.

## Hosting (as specified)
- **Frontends** (`adorbit-advertiser/`, `orbitearn-earner/`, `admin-console/`) → three separate Vercel projects (each folder is a static site, deploy independently).
- **Backend** (`backend/`) → one Render Web Service.
- **Database** → one Railway PostgreSQL instance.

## Deployment steps

1. **Database**: create a Railway PostgreSQL instance, then run `backend/schema.sql` against it once.
2. **Backend**: push `backend/` to its own git repo, deploy to Render as a Web Service (`npm start`). Set env vars from `backend/.env.example` — copy your real `DATABASE_URL` from Railway, generate a fresh `JWT_SECRET`, and add Flutterwave keys when ready. Note the resulting `https://xxxx.onrender.com` URL.
3. **Frontends**: in each of `adorbit-advertiser/`, `orbitearn-earner/`, and `admin-console/`, replace the `API_BASE_URL` constant near the bottom of each HTML file with your real Render URL, then deploy each folder as its own Vercel project.
4. **Promote an admin**: after registering your own account through AdOrbit or OrbitEarn's sign-up form, run in Railway's SQL console:
   ```sql
   UPDATE users SET role = 'admin' WHERE email = 'you@example.com';
   ```
   Then log into the Admin Console with that same email/password.

## Pricing reference (all defined in UGX, converted automatically — see `backend/currency.js`)

| Format | Advertiser pays | Earner/publisher gets |
|---|---|---|
| Video ad (per view) | 200 UGX | 50 UGX flat |
| Social post (flat) | 20,000 UGX | 5,000 UGX flat |
| Banner click | 112 UGX | 60% split (67.2 UGX) |
| Classified (flat/week) | 7,500 UGX | — |
| "Generate for me" content | +50% on top of the above | — |

Currencies: UGX, KES, TZS, RWF, ZAR, USD, EUR.

## Follower verification (TikTok/X — manual until a paid API is added)

The Admin Console's **Social Verification** tab lists pending TikTok and X
submissions grouped by platform. For each one, click one of three buttons:
- **Below Minimum → Reject**
- **100–500 / 1,000–5,000 → Micro** (can only take campaigns with ≤100 total views/units)
- **500+ / 5,000+ → Standard** (full access)

YouTube verifies automatically if `YOUTUBE_API_KEY` is set in Render's env vars.

The moment you get a paid scraping/API key for TikTok or X, open
`backend/util/socialVerify.js` — there are commented-out blocks showing exactly
where to plug it in. Once wired in, those platforms become automatic too and
stop appearing in the manual queue.

## Full file map

```
AdOrbit-Platform/
├── backend/                    → Render
│   ├── server.js
│   ├── db.js
│   ├── currency.js
│   ├── paymentGateway.js
│   ├── schema.sql
│   ├── package.json / .env.example / .gitignore
│   ├── middleware/ (auth.js, requireRole.js)
│   ├── util/ (tiers.js, socialVerify.js)
│   └── routes/ (auth.js, campaigns.js, tasks.js, payments.js, admin.js, earnerProfile.js)
├── adorbit-advertiser/          → Vercel project #1
│   ├── index.html, dashboard.html
│   └── assets/ (logo.svg, favicon.svg)
├── orbitearn-earner/            → Vercel project #2
│   ├── index.html, onboarding.html, dashboard.html
│   └── assets/ (logo.svg, favicon.svg)
└── admin-console/                → Vercel project #3
    ├── index.html, dashboard.html
```

## Still open (confirm with me before launch)

1. **X micro-tier ceiling**: 1,000–5,000 assumed by scaling TikTok/YouTube's
   100–500 range 10x. Confirm or adjust in `backend/util/tiers.js`.
2. **Real credentials**: rotate the DB/JWT/Flutterwave values that were in
   earlier `.env` files shared in this project — treat them as compromised.
3. Update every `API_BASE_URL` placeholder once your Render URL is live.
