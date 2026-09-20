# AdOrbit / OrbitEarn — Full Platform

A twin-platform advertising network:
- **AdOrbit** (`adorbit-advertiser/`) — advertisers create and pay for campaigns.
- **OrbitEarn** (`orbitearn-earner/`) — earners complete tasks and get paid.
- **Admin Console** (`admin-console/`) — content generation queue, social proof review, and social account verification.
- **Backend** (`backend/`) — one shared Node.js/Express API for all three.

## Hosting (as specified)
- **Frontends** (`adorbit-advertiser/`, `orbitearn-earner/`, `admin-console/`) → three separate Vercel projects (each folder is a static site, deploy independently).
- **Backend** (`backend/`) → one Render Web Service.
- **Database** → one Railway PostgreSQL instance.

## Deployment steps

1. **Database**: create a Railway PostgreSQL instance, then run `backend/schema.sql` against it once.
2. **Backend**: push `backend/` to its own git repo, deploy to Render as a Web Service (`npm start`) at `https://afriad-central-backend.onrender.com` (already wired into every frontend below — rename the Render service to match, or update `API_BASE_URL` in each HTML file if you use a different URL). Set env vars from `backend/.env.example` — copy your real `DATABASE_URL` from Railway, generate a fresh `JWT_SECRET`, and add Flutterwave keys when ready.
3. **Frontends**: deploy each of `adorbit-advertiser/`, `orbitearn-earner/`, and `admin-console/` as its own Vercel project. `API_BASE_URL` in every HTML file already points to `https://afriad-central-backend.onrender.com` — only change it if your Render service ends up at a different URL.
4. **Promote an admin**: after registering your own account through AdOrbit or OrbitEarn's sign-up form, run in Railway's SQL console:
   ```sql
   UPDATE users SET role = 'admin' WHERE email = 'you@example.com';
   ```
   Then log into the Admin Console with that same email/password.

## Pricing reference (all defined in UGX, converted automatically — see `backend/currency.js`)

| Format | Advertiser pays | Earner gets |
|---|---|---|
| Video ad (per tracked view) | 200 UGX | 50 UGX per view |
| Social post (flat — image or video creative) | 20,000 UGX | 5,000 UGX flat |
| Classified (flat/week) | 7,500 UGX | — |
| "Generate for me" content | +50% on top of the above | — |

Currencies: UGX, KES, TZS, RWF, ZAR, USD, EUR.

**No in-app "watch and earn," and no banner clicks.** There are exactly two
ways an earner makes money, and both require posting to their own account:

- **Video ads**: post the video to your own TikTok or YouTube, submit the
  live link (`POST /api/tasks/submit-video-proof`). From there the platform
  tracks the view count on that link over time and pays 50 UGX-equivalent
  per view as views accrue — see "View tracking" below.
- **Social posts**: post the creative (image or video) to TikTok, YouTube,
  or X, submit the live link (`POST /api/tasks/submit-social-proof`). An
  admin confirms it's really live, then the flat 5,000 UGX-equivalent clears
  from pending to withdrawable balance.

## View tracking (how video ads get paid)

Unlike social posts, a video ad's view count isn't a one-time check — it
climbs over time as the earner's post gets more views. So instead of a
single approve/reject, the Admin Console has a **Video View Tracking** tab
listing every active submission. An admin periodically checks each link's
current view count and enters it:

- **YouTube**: an "Auto-Check" button attempts to pull the real view count
  via the Data API (`YOUTUBE_API_KEY` required) and fills the field in.
- **TikTok**: manual — open the link, read the view count, type it in.

Only the *new* views since the last check are ever paid out (`routes/admin.js`'s
`/video-tasks/:id/update-views` computes the delta), so re-checking a link
is always safe and never double-pays. The delta is also capped by the
campaign's remaining budget, so a campaign can't be overspent even if
several earners are posting the same creative. Advertisers see live
progress — total views delivered and a per-earner breakdown with each link
and its running view count — on their dashboard (`GET /api/campaigns/:id/video-stats`).

When a paid TikTok view-count API is added later, open `backend/util/videoStats.js`
— there's a commented block showing exactly where to plug it in, after which
TikTok auto-checks the same way YouTube does.

## Follower verification (TikTok/X — manual until a paid API is added)

There's a single gate for earning on OrbitEarn — **no tiers**: at least one
verified TikTok/YouTube account with 100+ followers, or an X account with
1,000+ followers. Meet that once and you see every campaign matching your
niche and location, same as anyone else.

The Admin Console's **Social Verification** tab lists pending TikTok and X
submissions grouped by platform, each with **Approve** / **Reject** buttons
and an optional follower-count field for your own records. YouTube verifies
automatically if `YOUTUBE_API_KEY` is set in Render's env vars.

The moment you get a paid scraping/API key for TikTok or X, open
`backend/util/socialVerify.js` — there are commented-out blocks showing exactly
where to plug it in. Once wired in, those platforms auto-verify too and stop
appearing in the manual queue.

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
│   ├── util/ (eligibility.js, socialVerify.js, videoStats.js)
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

1. **Real credentials**: rotate the DB/JWT/Flutterwave values that were in
   earlier `.env` files shared in this project — treat them as compromised.
