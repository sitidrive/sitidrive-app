# MASTER PROMPT V2: Build SitiDrive (Full-Stack Pilot MVP)

You are a senior full-stack engineer. Build **SitiDrive** end to end from this ONE document. Everything you need (product, data model, backend, integrations, design, avatars, phases, acceptance tests) is in here. Do not invent services outside section 22.

Work in the phases in section 23. Finish and verify each phase before starting the next. At the end of each phase print: what you built, what you tested, what you assumed.

**Behaviour rules for you:** Ask nothing. If something is unspecified, choose the safest reasonable default, record it under "Assumptions" in `README.md`, and continue. Do not skip tests. Do not leave TODO stubs in money, seat, or security paths. Do not add features outside section 3.

---

## 0. WHAT CHANGED FROM V1 (read first; these override any older instruction)

| Topic | V1 | V2 (this file) |
|---|---|---|
| Backend | Firebase | **Supabase (Postgres + Auth + Edge Functions + Realtime), free tier.** Reason: Firebase Cloud Functions need a paid billing plan, Supabase free tier covers everything here. | Include doc stating how to connect and setup backend to these services before launch. name it 'backend setup.txt' it should contain clear instructions of fixing connecting the app to the backend and any external api services.
| Platform fee | 7% | **5%** (`app_config.commission_rate = 0.05`, never hard-coded) |
| Design | Sky-blue palette | **New brand system from the supplied logo and avatars** (charcoal, orange, teal, magenta, coral). Layout patterns come from the 4 supplied mobile mockups; their blue colours and sample route names are NOT used. |
| New feature | none | **Live "waiting riders" and "available rides" interactive icons** (section 12) |
| New feature | none | **Admin-created Events** where drivers offer and riders board rides to and from a venue (section 13) |
| Integrations file | separate | **Merged into section 22** |
| Avatars | none | **Avatar system** (section 18, sprite in Appendix A) |
---

## 1. PRODUCT

SitiDrive is a **route-based shared commuting platform** piloting in **Abuja, Nigeria**. Riders and drivers share a car (max **4 passengers**) along predefined corridors with ordered stops, picking up passengers going the same way, so each rider pays a fraction of a normal cab fare.

It is **NOT** private point-to-point ride-hailing. Everything is organised around: routes, ordered stops, departures (offered rides), seat capacity, reservations, and (new) events.

**Pilot objective:** prove SitiDrive can coordinate recurring shared journeys on defined Abuja corridors and at big events, raise seat utilisation, and support sustainable economics for riders, drivers, and the platform (5% fee).

**Brand voice:** premium, youthful, trustworthy, urban, Fun, Community. Functional minimalism. Splash and onboarding tagline (uppercase, wide letter-spacing): **SHARED CITY RIDES. SMARTER COMFY COMMUTES.**

---

## 2. FIRST-PRINCIPLES RULES (these override convenience)

1. **The client is hostile.** Anything touching money, seats, points, roles, ratings, or trip status is written ONLY by trusted backend code (Postgres RPC functions marked `SECURITY DEFINER`, and Edge Functions). The browser has **no** INSERT/UPDATE/DELETE grants on any table. All writes go through named RPCs.
2. **Single source of truth per fact.** Seats, fares, points, commissions are each computed in exactly one backend place (a SQL function). The TypeScript mirror is for live preview only and is tested against the same fixtures.
3. **Append-only ledgers for value.** `points_ledger`, `commissions`, `trip_events`, `audit_log` cannot be updated or deleted (enforced by trigger that raises). Balances change only inside the same transaction that inserts the ledger row.
4. **Operational truth.** Never show a departure as available without a valid availability record. Never invent live positions, ETAs, driver verification, seat counts, rider counts, or reviews. If live data is unavailable, show scheduled or last-confirmed status. "Waiting riders" counts come only from real active `waiting_posts` rows.
5. **Delete what is not needed.** If it is not in section 3, do not build it.
6. **Physics first:** booking, seats, fare split, payment, commission. Polish second.
7. **Money is integer naira.** No floats. Convert to kobo (x100) only at the Paystack boundary.

---

## 3. MUST-HAVE FEATURES (scope is exactly this)

| # | Feature |
|---|---|
| 1 | **Google sign-in only** (Supabase Auth, Google provider). No email/password, no Apple. | include how to setup connect step by step guide.
| 2 | **Profile for every user**: avatar, nickname, **Points box** (coin icon, balance, history). |
| 3 | **Book a ride** (rider flow). |
| 4 | **Offer a ride** (driver flow).include driver gender,email, plate number and these are saved and sent to the backend for compliance and security reasons |
| 5 | **Referral / share app**: each successful referral earns points. |
| 6 | **Squad**: visible but locked, "Coming soon" teaser. |
   7   **Solo**: visible but locked, "Coming soon" teaser. |
| 8 | **Fare engine**: whole-vehicle trip fare split across up to 4 riders, live in the UI. |
| 9 | **Driver onboarding** with admin approval. |
| 10 | **Payments**: riders pay driver by cash or transfer (driver toggles); **driver pays 5% platform fee to us via Paystack**. |
| 11 | **Every trip tracked categorised into complete and initiated** in the database and visible in Admin.  and a tack of drivers with the most rips completed so we can reward them|
| 12 | **Admin dashboard**, restricted to two admins. |
| 13 | **Dispute resolution**, backend and admin only, no rider/driver UI. |
| 14 | **Safety reminder popups** ask rider to be vigilant confrm their ride and driver to make sure they always park corectly and SOS. |
| 15 | **Wow landing page.** |
| 16 | **Hardened RLS and security posture.** |
| 17 | **Fully responsive, mobile-first** from 320px to 4K. |
| 18 | **NEW: Live interactive icons**: see available rides and waiting riders on the route map (section 12). |
| 19 | **NEW: Admin-created Events** with their own routes, banner, and ride board (section 13). |
| 20 | **NEW: Avatar system** (section 18). |

### Explicitly DO NOT BUILD
- Apple sign-in, email/password
- "Popular Routes" carousel (use real corridors and live events instead),"Quick Access" Home/Work/School shortcuts
- Demo seed data (other than the 4 route terminals and the config row), "wipe logs", "total reset", destructive admin actions
- Role-switcher debug mode; hard-coded personal emails in client code
- `Math.random()` map centres, fake loading animations, fake riders or drivers
- Automatic next-ride matching, GPS auto-tracking, heatmaps (V2 product). Waiting posts are stop-level only.
- Client-side writes of ratings, payment status, seats, points
- Web3/tokens (points are plain numbers)
- In-app chat, in-app rider card payments (flag exists, default OFF, see 11.7)

---

## 4. BRAND ASSETS AND DESIGN SYSTEM

### 4.1 Assets the founder supplies (place exactly here; do not redraw)
```
public/brand/logo.png            full wordmark "sitidrive"
public/brand/favicon.png         512x512 app icon (rounded charcoal square, orange circle + white bar "d", 3 coloured dots)
public/avatars/sitidrive-avatars-sprite.svg   (content in Appendix A)
public/avatars/tokens.json                    (content in Appendix B)
```
Generate from `favicon.png`: `favicon.ico`, 16/32/180/192/512 PNGs, `apple-touch-icon.png`, maskable icon, `site.webmanifest` (theme colour `#1F2937`, background `#FBF8F3`), and the Open Graph image (1200x630: logo on `#FBF8F3` with the tagline). If a file is missing at build time, render a clean text wordmark "sitidrive" in the brand font and log a warning; never fail the build.

**Logo usage:** wordmark = "siti" + "drive" in charcoal with an orange circle plus charcoal bar forming the "d", coral/magenta/teal square dots over the i's, and a teal "e". Keep clear space equal to the height of the dots. Minimum width 96px. On dark backgrounds use a white-text variant (CSS filter fallback is acceptable until a dark logo is supplied). Never recolour, stretch, or rotate. Use the favicon square for app icon, PWA icon, and the compact header mark on mobile scroll.

### 4.2 Colour tokens (CSS variables + Tailwind theme, also written to `DESIGN.md`)
```
--sd-charcoal:#1F2937  --sd-ink:#111827
--sd-orange:#F97316    --sd-orange-soft:#FFEDD5  --sd-orange-deep:#C2410C
--sd-teal:#14B8A6      --sd-teal-soft:#CCFBF1    --sd-teal-deep:#0F766E
--sd-magenta:#EC4899   --sd-magenta-soft:#FCE7F3
--sd-coral:#F87171     --sd-coral-soft:#FEE2E2   (alerts, SOS)
--sd-gold:#FBBF24      (coins, points only)
--sd-bg:#FBF8F3  --sd-card:#FFFFFF  --sd-line:#E5E7EB  --sd-muted:#6B7280
Dark mode: --sd-bg:#0F172A  --sd-card:#1E293B  --sd-line:#334155 (avatars unchanged, 1px #FFFFFF1A ring)
Semantic: success=teal, warning=gold, error=coral-deep #DC2626
```
**Contrast rules (WCAG AA):** white text on orange fails, so primary buttons are **charcoal with white text**; orange is for accents, icons, highlights, and orange buttons use **charcoal text**. Orange text on white only at 24px+ bold or use `--sd-orange-deep`.

Type: Inter (UI) and JetBrains Mono (plates, fares, times), **self-hosted via @fontsource** (keeps CSP strict). 8pt grid, radius 12 to 24px, touch targets >= 44px, primary actions in the bottom 30% thumb zone, spring animations for sheets, honour `prefers-reduced-motion`.
Payment badges: 💵 cash, 📱 transfer, with a tiny bounce on arrival.

### 4.3 Screen patterns taken from the supplied mockups (structure only)
Rebuild these layouts in the new palette. **Ignore the mockups' sample route names (Kubwa, Wuse, Maitama, Gwarinpa, Life Camp), sample prices, "est. time", and stop times: they are placeholders. Use only the routes in section 5.**

| Screen | Structure |
|---|---|
| Splash / onboarding | Logo, hero illustration, row of 5 avatars, tagline, 3-dot pager, **Continue with Google** button, no email option |
| Home | Logo + bell + avatar (top), greeting "Where are you heading today?", search field (routes/stops), two big buttons **Find a Ride** / **Offer a Ride** (Offer shown only if user has driver role or can apply), **Live Events** carousel (only when events exist), **Corridors** cards (the 4 real routes with seats-left from real departures), Squad locked card, bottom nav |
| Bottom nav | Home, Routes, Squad (lock badge), Trips, Profile |
| Book a Ride | Tabs **Available Routes / My Bookings**, search, filter, route cards with live departures count |
| Route Details | Header band (original flat skyline illustration in brand colours), "Fixed Route" pill, route name with arrow, row of stats (**seats left** and **per-rider fare now**, both from real data; show typical duration ONLY if admin entered it), ordered stop timeline (times only if admin entered stop offsets), "Approved by SitiDrive" trust card, **Book Seat** button |
| Departure list (new) | Cards with driver avatar, plate hidden until confirmed, payment badges, seat pips ●●○○, live fare "Now ₦X each. Drops to ₦Y at 4 riders" |
| Ticket (new) | Large monospace plate, driver avatar (xl), co-rider avatar stack, payment method, safety card, SOS |

Illustrations: draw original flat SVG skylines and vehicle shapes in the brand palette. Do not copy the mockup artwork.

---

## 5. DOMAIN: ROUTES (founder-supplied; admin-editable; do NOT invent intermediate stops)

- **Route A:** MarketSquare Bridge → Central Area (key destination: Federal Secretariat)
- **Route B:** Federal Ministry of Finance / Finance Bridge → Lugbe
- **Route C:** Federal Ministry of Finance / Finance Bridge → Banex
- **Route D:** Area 11 Junction → Nyanya

Seed ONLY an origin stop and a terminal stop per route (sequence 1 and 2). Coordinates are entered by admin in the admin UI (stops are created `active=false` until lat/lng are set; inactive stops cannot be booked). Admin adds intermediate stops later (drag to reorder; `sequence` unique per route). Also seed one `app_config` row.

---

## 6. DATABASE (Postgres, via `supabase/migrations/*.sql`)

All tables have RLS **enabled and forced**. `REVOKE ALL ON ALL TABLES IN SCHEMA public FROM anon, authenticated;` then grant SELECT only where listed in section 21. Money columns are `integer` naira. IDs are `uuid default gen_random_uuid()`. All timestamps `timestamptz default now()`.

```
profiles(id uuid pk references auth.users, display_name text, email text, photo_url text, avatar_id text,
  phone text, roles text[] default '{rider}', points_balance integer default 0 check(points_balance>=0),
  referral_code text unique not null, referred_by uuid, created_at, last_active)

admin_allowlist(email text pk)                  -- no client access at all
admins(user_id uuid pk)                         -- filled by trigger on auth.users when verified Google email is in admin_allowlist
function is_admin() security definer            -- exists(select 1 from admins where user_id=auth.uid())

driver_profiles(user_id pk, status text check in('pending','approved','rejected','suspended'),
  full_name, plate_number, vehicle_type, accepts_cash bool, accepts_transfer bool,
  check(accepts_cash or accepts_transfer), submitted_at, reviewed_by, reviewed_at)
driver_bank(user_id pk, bank_name, account_number char(10), account_name, updated_at)   -- strict RLS (see 21)

events(id, title, description, venue_name, venue_lat, venue_lng, starts_at, ends_at, banner_url,
  status check in('draft','published','cancelled','ended'), publish_at, bonus_points_per_trip int default 0,
  created_by, created_at)

routes(id, name, direction text, kind check in('corridor','event') default 'corridor', event_id references events null,
  active bool, pricing_policy check in('flat','pair'), flat_trip_fare int, typical_duration_min int null)
stops(id, route_id, name, sequence int, lat, lng, active bool, offset_min int null, unique(route_id,sequence))
route_fares(route_id, pickup_stop_id, drop_stop_id, trip_fare int, primary key(route_id,pickup_stop_id,drop_stop_id))

rides(id, route_id, driver_id, departure_time, status check in('scheduled','locked','ongoing','completed','cancelled'),
  capacity int check(capacity between 1 and 4), seats_booked int[] not null,   -- length = stops-1 (segments)
  plate_snapshot, vehicle_snapshot, accepts_cash, accepts_transfer, created_at)

bookings(id, ride_id, rider_id, pickup_stop_id, drop_stop_id, pickup_seq int, drop_seq int, seats int check(seats between 1 and 4),
  status check in('requested','confirmed','declined','cancelled','completed','no_show'),
  payment_method check in('cash','transfer'),
  payment_status check in('pending','marked_paid','confirmed','disputed') default 'pending',
  payment_reference text, trip_fare_snapshot int, fare_share_at_booking int, final_fare int, created_at,
  check(pickup_seq < drop_seq))

waiting_posts(id, rider_id, route_id, stop_id, seats int default 1, window_start, window_end,
  status check in('active','cancelled','expired'), expires_at, created_at)

points_ledger(id, user_id, delta int, reason text, ref_id uuid, created_at)            -- append-only
commissions(id, ride_id unique, driver_id, gross_fare int, commission_rate numeric, commission_amount int,
  status check in('owed','paid','waived'), paystack_reference text unique, due_at, paid_at, created_at)
paystack_events(reference text pk, event text, payload jsonb, received_at)               -- webhook idempotency
referrals(id, referrer_id, referred_id unique, status check in('pending','rewarded','rejected'), created_at, rewarded_at)
ratings(id, booking_id unique, driver_id, rider_id, stars int check between 1 and 5, created_at)
disputes(id, booking_id, ride_id, raised_by text('system','admin'), reason, status check in('open','investigating','resolved','dismissed'),
  notes jsonb default '[]', resolution text null, created_at, resolved_at)
trip_events(id, ride_id, booking_id null, type, actor_id, payload jsonb, created_at)     -- append-only
audit_log(id, actor_id, action, target, payload jsonb, created_at)                       -- append-only (bank edits, admin actions)
notifications(id, user_id, kind, title, body, ref_id, read_at null, created_at)           -- in-app, Realtime
analytics_events(id, type, user_id, meta jsonb, created_at)
squad_waitlist(user_id pk, created_at)
rate_limits(user_id, bucket text, window_start, count)                                    -- used inside RPCs
app_config(id int pk default 1 check(id=1), commission_rate numeric default 0.05, max_seats int default 4,
  points_per_completed_trip int, referral_bonus int, daily_points_cap int,
  overdue_threshold_naira int default 5000, overdue_days int default 7,
  request_hold_minutes int default 15, waiting_post_ttl_minutes int default 120,
  min_fare_floor_enabled bool default false, min_fare_floor int null,
  event_grace_hours int default 3, emergency_numbers jsonb default '[]', safety_copy jsonb, push_enabled bool default false)
```
Required indexes: `rides(route_id,status,departure_time)`, `bookings(ride_id,status)`, `bookings(rider_id,created_at desc)`, `waiting_posts(route_id,stop_id,status,expires_at)`, `commissions(driver_id,status)`, `points_ledger(user_id,created_at desc)`, `trip_events(ride_id,created_at)`, `stops(route_id,sequence)`.
Enable **Realtime** on: `rides`, `waiting_posts` (via the safe views), `bookings`, `notifications`, `points_ledger`, `events`. Realtime respects RLS.

---

## 7. CORE RULES (all in RPCs; never trust the UI)

### 7.1 RPC catalogue (the only write path; each checks `auth.uid()`, validates input, rate-limits per user, writes `trip_events`/`audit_log` where relevant)
```
Profile:   ensure_profile(), update_my_profile(display_name, phone, avatar_id, photo_url), touch_last_active()
Referral:  redeem_referral(code)
Driver:    submit_driver_application(full_name, plate, vehicle_type, accepts_cash, accepts_transfer, bank?)
           update_driver_settings(accepts_cash, accepts_transfer), update_driver_bank(...)   -- writes audit_log
Rides:     offer_ride(route_id, departure_time, capacity), lock_ride(id), start_ride(id), complete_ride(id), cancel_ride(id, reason)
Bookings:  request_booking(ride_id, pickup_stop_id, drop_stop_id, seats, payment_method), respond_to_request(booking_id, accept bool),
           cancel_booking(booking_id), mark_paid(booking_id, reference?), confirm_payment(booking_id, received bool),
           mark_no_show(booking_id), submit_rating(booking_id, stars)
Waiting:   post_waiting(route_id, stop_id, seats, window_start, window_end), cancel_waiting(id)
Squad:     join_squad_waitlist()
Admin:     admin_review_driver(user_id, decision), admin_upsert_route/stop, admin_reorder_stops(route_id, ordered_ids[]), admin_set_fare(...),
           admin_upsert_event, admin_publish_event, admin_end_event, admin_cancel_event, admin_broadcast_event(event_id),
           admin_add_dispute_note, admin_resolve_dispute(id, outcome), admin_set_commission_status(id, 'waived'), admin_update_config(patch),
           admin_export_points_csv()
```
Edge Functions: `paystack-init-commission`, `paystack-webhook`, `send-web-push` (flag, default OFF).

### 7.2 Stop validation
`pickup_seq < drop_seq` on the same route; both stops `active`. A rider may alight at any active intermediate stop; the terminal ends the route but is not every rider's destination. Reject anything else.

### 7.3 Seat inventory (concurrency-safe)
`rides.seats_booked[i]` = seats held on segment i (between stop i and i+1).
`request_booking` runs in one transaction: `SELECT ... FROM rides WHERE id=$1 FOR UPDATE`; verify ride `scheduled`; verify for every segment from pickup_seq to drop_seq-1 that `seats_booked[i] + seats <= capacity`; insert booking `requested`; increment the array. **Requested bookings hold seats** (so riders cannot request a full car). Decline, cancel, hold expiry (`request_hold_minutes`, run by `pg_cron`), or ride lock restores the seats. Confirmed seats can never exceed capacity. Rider may have only one active booking per ride.
`cancel_booking` is allowed until the ride is `locked`; afterwards only admin may cancel.
**Concurrency test required:** 10 parallel `request_booking` calls for the last seat produce exactly 1 success.

### 7.4 Ride lifecycle (backend-controlled)
`scheduled → locked → ongoing → completed`, or `→ cancelled` (from scheduled or locked). Only the owning driver (or admin) can transition, and every transition inserts a `trip_events` row. `lock_ride` fails while any booking is still `requested` (driver must accept or decline all first). `lock_ride` freezes `final_fare` on every confirmed booking (section 8). `complete_ride` (one transaction): marks confirmed bookings `completed`, creates the commission (section 11), awards points (section 17), writes events. `start_ride` is allowed only from `locked`.
Departure must be in the future when offered. A driver cannot have two overlapping rides.

### 7.5 Pre-lock requests
Riders appear as **requests** on the driver screen with a payment-method badge, rider avatar and first name. Accept → `confirmed`; decline → `declined` (seats released, rider notified).

---

## 8. FARE ENGINE

Fares come from explicit admin config. **Never invent prices.** A route with no fare config is not bookable.
- `pricing_policy='flat'` → `trip_fare = flat_trip_fare`.
- `'pair'` → `trip_fare` from `route_fares(pickup,drop)`; missing pair = not bookable.
- `trip_fare` is the **whole-vehicle fare** for that journey.

```
n   = number of confirmed riders (seats) on the booking's busiest overlapping segment, 1..4   (min 1)
perRiderShare = ceil(trip_fare / n)
```
- Implement as SQL function `compute_rider_share(trip_fare, n)` (authoritative) and a TS mirror `fare.ts`. Both are tested against the same JSON fixtures (`tests/fixtures/fare-vectors.json`, include n=1..4, odd fares, rounding, floor flag on and off).
- Optional floor: if `min_fare_floor_enabled`, `share = max(share, min_fare_floor)`. Default OFF (open founder decision).
- Before lock the UI shows live: **"Now ₦X each. Drops to ₦Y when 4 riders join."** It animates as confirmed riders join or leave (Realtime).
- At `locked`, each booking's `final_fare` is frozen; later fare config changes never alter it.
- Driver total = sum of `final_fare`. Platform fee = 5% of that sum.
- Assumption to record in README (founders to confirm): riders with different pickup/drop pairs on the same ride are each priced from their own `trip_fare` and the busiest-overlap `n`.

---

## 9. DRIVER ONBOARDING

Fields: full name, plate, vehicle type, Google email (from auth), payment options (cash/transfer), bank details (if transfer but cash advisable).
- `submit_driver_application` sets `status='pending'`; the application appears in Admin → Drivers. Admin approves, rejects, or suspends. Only `approved` drivers can `offer_ride`.
- **Bank details** show on the driver profile and on the ticket only to riders holding a confirmed booking on that driver's ride where transfer is accepted (enforced by RLS on `driver_bank`, and via an RPC-backed view). Editable any time; each edit writes `audit_log`. Never visible to anonymous users.
- Driver dashboard toggles "I accept cash" / "I accept transfer" (at least one on). Each ride snapshots the toggles at offer time.
- Validation (Zod on client + SQL checks): plate trimmed and uppercased, loose Nigerian format (letters/digits, 5 to 10 chars), 10-digit NUBAN account number.

---

## 10. SAFETY AND TRUST

- **Driver, before going live and before each ride:** "Park in a safe, visible spot. Check your riders. Respect every rider and keep the ride fun."
- **Rider, on confirmation and at pickup:** "Verify the plate number **{plate}** before you board. Pay your driver as agreed. Be respectful and enjoy the ride."
- Plate shown large in monospace on the rider's ticket (only after the booking is confirmed).
- **SOS button** on the rider ride screen. Numbers come from `app_config.emergency_numbers` (admin enters them). If empty, the admin dashboard shows a blocking "Set emergency numbers before launch" banner and the README launch checklist lists it. Never hard-code numbers.
- "Approved by SitiDrive" wording only for drivers with `status='approved'`. Do not claim background checks unless admin config says so.
- All copy editable in `app_config.safety_copy`.

---

## 11. PAYMENTS AND THE 5% PLATFORM FEE

Riders pay the **driver directly** by cash or bank transfer. SitiDrive earns **5% of each completed ride's total fare**, collected through our **Paystack** account.

**Mechanism (implement exactly):**
1. `complete_ride` creates one `commissions` row: `commission_amount = round(commission_rate * sum(final_fare of completed bookings))` using `app_config.commission_rate` (default 0.05), `status='owed'`, `due_at = now() + overdue_days`.
2. Driver dashboard shows "Platform fee owed: ₦X" and a **Pay now** button. It calls Edge Function `paystack-init-commission(commission_ids[])`, which verifies the caller owns those rows, builds a unique reference, calls Paystack `transaction/initialize` (amount in kobo, email = driver email, `callback_url` = app), stores the reference on the rows, and returns `authorization_url`. The client redirects to Paystack Checkout (card, bank, transfer, USSD). **The secret key never reaches the browser.**
3. `paystack-webhook` Edge Function: read the raw body, verify header `x-paystack-signature` = HMAC-SHA512 of the raw body with the Paystack secret key (constant-time compare), reject otherwise. On `charge.success`, also call Paystack `transaction/verify/:reference` and confirm status `success`, currency NGN, and amount equals the sum owed. Then mark the commissions `paid`. **Never trust the client redirect as proof.**
4. **Idempotent:** insert the reference into `paystack_events` first (primary key); if it already exists, return 200 and do nothing.
5. **Enforcement:** `offer_ride` rejects drivers whose unpaid total exceeds `overdue_threshold_naira` (default ₦5,000) or who have any commission past `due_at`. The UI explains why and offers Pay now.
6. Admin → Money: owed, paid, overdue, total earned, Paystack references, waive action (audited).
7. **Flag, default OFF:** in-app rider card payment with Paystack split payments (subaccount) where the platform keeps the fee automatically. Build only the config flag and a stub that returns "not enabled".
8. Paystack's own processing fee is absorbed by the platform by default (`pass_processing_fee_to_driver` flag, default OFF).

**Booking payment status flow:** `pending → marked_paid` (rider taps "I've paid", optional reference) `→ confirmed` (driver confirms). Driver disagreement → `disputed` and a system dispute is created. Riders cannot self-confirm. Cash riders also tap "I've paid" at the end of the ride; the driver confirms.

---

## 12. LIVE INTERACTIVE ICONS: AVAILABLE RIDES AND WAITING RIDERS (NEW)

Purpose: let riders see rides that exist, and let drivers see real demand, so people can find each other along the fixed corridors. **Stop-level only. No GPS tracking, no live positions.**

### 12.1 Waiting riders ("I'm waiting here")
- On a route screen a rider taps a stop and **"I'm waiting here"**, choosing a time window (default now to +60 min, max `waiting_post_ttl_minutes`) and seats (1 to 4). RPC `post_waiting`. One active post per user per route. Posts expire automatically (`pg_cron` + `expires_at` filtering). Rider can cancel any time. Rate limited.
- Anyone signed in sees an **aggregated, privacy-safe** view `stop_demand` (security definer): per route and stop, `waiting_count`, up to 3 `avatar_id`s, and first names only. Never rider ids, phone, email, or exact times beyond the window.

### 12.2 Interactive icons on the map and lists (Leaflet markers via `L.divIcon`, keyboard accessible)
- **Stop pin with avatar bubbles** + count badge (e.g. "3 waiting"). Tap opens a bottom sheet: waiting avatars (stack), window, and for drivers a primary button **"Offer a ride from here"** that pre-fills `offer_ride` (route, time window start). For riders the sheet shows upcoming departures from that stop.
- **Car icon per scheduled departure** at its origin stop, with **seat pips** ●●○○ (real remaining capacity on the relevant segment) and departure time. Tap opens the departure sheet with fare split, payment badges, and **Request seat**.
- Icons update live via Realtime with a gentle pop animation (disabled for reduced-motion). Use cluster/offset so overlapping markers stay tappable at 320px.
- Empty states are honest: "No one waiting here yet. Be the first." Never show placeholder people.
- When a driver publishes a ride on a route, riders with an active waiting post on that route/stop get an in-app notification "A ride is offered at your stop at HH:MM" (notification only; they still book themselves).
- Same components power event routes (section 13).

---

## 13. EVENTS (NEW)

Admin can spin up a major event (concert, conference, match, church or market day) so drivers can offer rides and riders can board rides to and from it.

### 13.1 Data and rules
- `events` (section 6). Lifecycle: `draft → published → ended` or `cancelled`. Only `published` events whose window is open (from `publish_at` to `ends_at + event_grace_hours`) are visible and bookable.
- Each event owns one or two **event routes** (`routes.kind='event'`, `event_id` set): **To event** (hub stops → venue) and optionally **From event** (venue → hub stops). Admin defines the hub stops (name + lat/lng via map click) and the venue stop (defaults to `venue_lat/lng`). **Do not invent hubs.** Fares are admin-set only (flat or pair), same engine as corridors.
- Event routes use the exact same machinery: offering (`offer_ride`), requests, fare split, lock, payments, 5% fee, points, disputes, waiting posts, interactive icons. `offer_ride` and `request_booking` additionally check that the event is live. Rides cannot depart after `ends_at + grace`.
- Optional `bonus_points_per_trip` (default 0) added to points on completed event trips.
- When an event ends or is cancelled: open rides on its routes are cancelled by backend (bookings cancelled, riders and drivers notified, seats released, `trip_events` written). Completed trips and commissions stay.

### 13.2 Admin "Events" tab (wizard)
1. Details: title, description, venue, date/time, banner image (upload to Supabase Storage bucket `event-banners`, public read, admin write, 2MB, image types only).
2. Routes and stops: add hub stops and venue on a Leaflet map (click to place, drag to reorder), choose direction(s).
3. Fares: flat or pair.
4. Preview as rider and as driver, then **Publish now** or schedule `publish_at`.
5. Live view: rides offered, seats filled, waiting riders per hub, fees owed, with **End event** and **Cancel event** (confirm dialog, audited).
6. **Broadcast:** one in-app notification to all users (and web push if enabled) via `admin_broadcast_event`, limited to once per event unless admin confirms resend.

### 13.3 User-facing
- Home: **Live Events** carousel (banner, title, date, "X rides, Y waiting", real counts), only when events exist.
- Event page: header, venue map with hub pins, tabs **Go to event / Back home**, the interactive icons (section 12), driver CTA **Offer a ride to this event**, rider CTA **Find a ride**, safety copy.
- Landing page "Going somewhere big?" section shows live events (hidden when none).
- Pulse tab shows event rides separately.

---

## 14. SQUAD (locked teaser)
- Visible **Squad** card on rider home and bottom nav with a lock icon and "Coming soon".
- Tap opens a polished bottom sheet: friendly illustration of friends, family and coworkers in one car; copy "Ride together. Split together. Squads for friends, family and coworkers. Coming soon."; **Notify me** button calls `join_squad_waitlist()` (one row per user; button shows "You're on the list").
- No functional squad logic.

---

## 15. DISPUTE RESOLUTION (backend and admin only; NO rider/driver UI)
- Created by backend only: rider marked paid and driver rejected; no-show with payment marked; ride cancelled after lock with payment marked; event cancelled after bookings paid.
- Admin → Issues: list, filter by status, view booking, ride, and `trip_events` timeline, add notes, resolve with outcome `rider_favoured | driver_favoured | split | dismissed`, which may adjust points or waive a commission through backend functions. Every action writes `trip_events`/`audit_log`. Disputes are immutable except via admin RPCs.
- Users get a plain in-app notification of outcomes; they cannot raise or edit disputes.

---

## 16. ADMIN DASHBOARD
**Access: ONLY allowlisted verified Google emails**: `francisetham01@gmail.com` (founder) and co-founder **Oma Merit** (email added later via the `admin_allowlist` table or `ADMIN_EMAILS` secret, **no code change**).
- A trigger on `auth.users` inserts into `admins` when the verified Google email is in `admin_allowlist`. Route guard calls `is_admin()` on the server; the client never decides. No emails in client bundles.
- Sign-in captcha (Cloudflare Turnstile) is enabled in Supabase Auth.

**Tabs (lean, all paginated, indexed queries only, no full scans):**
1. **Pulse:** total rides, completed rides, active riders and drivers, bookings last 7 days (Recharts line), seat utilisation %, 7-day retention, event rides, riders waiting now.
2. **Trips:** every ride live (status, driver, plate, riders, fare, payment state), filter/search, drill into event timeline.
3. **Drivers:** pending queue (name, plate, vehicle type, email), approve / reject / suspend.
4. **Routes:** routes, ordered stops (drag to reorder, unique `sequence`), coordinates, fare config, optional stop offsets and typical duration.
5. **Events:** wizard and live view (section 13).
6. **Money:** commissions owed / paid / overdue, Paystack references, waive, total earned.
7. **Points and Referrals:** ledger per user, referral funnel, top referrers, **CSV export** of balances.
8. **Issues:** disputes.
9. **Admins and Config:** read-only list of admin emails with instructions to update the allowlist; editable `app_config` (commission rate, points, thresholds, safety copy, emergency numbers).

---

## 17. POINTS, REFERRALS, PROFILE
- **Profile:** avatar (change via picker), nickname, email, phone, role badges, **Points box** (gold coin icon, large balance with animated count-up, history list), referral code with share button (Web Share API + copy link `?ref=CODE`), driver section (plate, vehicle, accept toggles, bank details editable, platform fee owed).
- **Points per trip:** `points_per_completed_trip` from config (+ event bonus), awarded inside `complete_ride` for the rider and driver (separately configurable), written to `points_ledger`, `profiles.points_balance` updated in the same transaction. Daily cap `daily_points_cap`.
- **Referral:** capture `?ref=` into `localStorage` (survives the Google redirect), call `redeem_referral` after first sign-in. Referrer is rewarded (`referral_bonus`) only when the referred user **completes their first trip**. One reward per referred user, self-referral blocked, status `rejected` on violation.
- **Popup on earn:** Realtime listener on the user's `points_ledger` shows a coin-burst toast "+50 points!". Subtle "future rewards" copy only. No tokens.
- Anti-fraud: points only from functions, only after real completed trips with confirmed payment, rate limits, sanity checks (same device/account patterns logged to `analytics_events` for admin review).

---

## 18. AVATAR SYSTEM
Use `public/avatars/sitidrive-avatars-sprite.svg` (Appendix A) and `tokens.json` (Appendix B). **Inline the sprite once at the app root** (cross-file `<use>` fails in some WebViews).
- Components: `Avatar`, `AvatarStack` (max 3 visible, -10px overlap at md, 2px border in app background, teal "+N" pill), `AvatarPicker` (selectable grid, selected = 3px teal ring), all driven by the `tokens.json` manifest so more avatars can be added by dropping SVGs and manifest entries with **no code change**. The founders have a 20-avatar (10 women, 10 men) expansion; **only the 10 in Appendix A are supplied here**, so ship with those and make the picker render however many the manifest lists. Do not tie avatar to gender.
- Sizes xs 24, sm 32, md 40, lg 56, xl 96, always circular. Assign `avatar_id` on first sign-in by stable hash of user id; user can change it in Profile. Fallback = initials on a soft tint (charcoal text). Driver = 2px orange ring **plus a small car badge** (never colour alone). Use the user's name as accessible label. Dark mode: unchanged avatars with a 1px `#FFFFFF1A` ring.
- Placement: onboarding row, home top-right, departure cards (driver lg + stack md), ticket (driver xl), requests list, waiting bubbles, ratings, empty states, Profile picker.
- Tests: initials fallback, "+N" overflow, aria-label, readable at 24px.

---

## 19. LANDING PAGE (WOW, HIGH-CONVERT) in `features/landing`
Pillars: *Split the ride, keep the ride. Up to 75% off a private cab fare when four riders share (always "vs. riding alone"). Approved drivers. Real Abuja corridors.*
Sections in order:
1. **Hero:** full-bleed, headline "Abuja, shared. Pay a quarter.", subhead, CTAs **Find a ride** / **Offer a ride** (both go to Google sign-in), animated 4-seat cabin where seats fill and the per-rider price ticks down (this is the product demo). Logo in header.
2. **Space for Four:** interactive cabin (1 front + 3 rear), tap seats to see the split change.
3. **Corridors:** the four real routes as cards or a Leaflet map with route lines.
4. **Going somewhere big?** live events (data-driven, hidden when none).
5. **How it works:** pick route and stops, join a ride, split and ride.
6. **Why SitiDrive:** cheaper, approved drivers and plate check, cash or transfer, earn points.
7. **Points and referrals teaser** with coin visual.
8. **Squad teaser** (locked).
9. **Social proof slot** (data-driven; no fabricated reviews).
10. **Final CTA + footer** (logo, links, "Built for African cities").
Performance and conversion: LCP < 2.5s on 4G, lazy-load below fold, sticky mobile CTA, SEO meta, Open Graph image, AA accessibility, reduced-motion safe.

---

## 20. TECH STACK AND STRUCTURE
- **Frontend:** React 19 + TypeScript (strict) + Vite, Tailwind CSS, shadcn/ui, Framer Motion (`motion/react`), React Router, Recharts (admin only), TanStack Query, Zod, `@supabase/supabase-js`.
- **Maps:** Leaflet + react-leaflet + OpenStreetMap tiles (no paid keys; keep attribution; OSM tile policy is fine for a small pilot, note in README to move to a tile provider before scale).
- **Backend:** Supabase (Postgres, Auth with Google, Realtime, Storage, Edge Functions in Deno/TypeScript, `pg_cron`).
- **Payments:** Paystack (secret key only in Edge Function secrets).
- **Hosting:** Cloudflare Pages or Vercel (free), with security headers file.
- **Testing:** Vitest, Playwright (smoke + responsive), Supabase local stack (`supabase start`) for RLS and RPC tests with two or more real JWT users.
```
src/ app/(router, providers, guards)  features/{auth,rider,driver,events,waiting,squad,profile,referral,points,admin,landing,avatars}
     components/ui  lib/(supabase.ts, format.ts, fare.ts)  hooks types styles
supabase/ migrations/ functions/{paystack-init-commission,paystack-webhook,send-web-push}  seed.sql  tests/
public/brand public/avatars
DESIGN.md README.md .env.example vercel.json|_headers  .github/workflows/ci.yml
```

---

## 21. SECURITY (RLS and platform)

**RLS policies (default deny; write real policies and test them):**
- `profiles`: owner SELECT own row; others see only through a `public_profiles` view exposing `id, display_name (first name only), avatar_id`. No direct UPDATE (use `update_my_profile`, which touches only display_name, phone, avatar_id, photo_url, last_active). `roles`, `points_balance`, `referral_code`, `referred_by` are never client-writable.
- `driver_profiles`: driver reads own; admin all; public sees only `status='approved'` via `public_drivers` view (first name, avatar, vehicle type, rating average). Plate visible only to riders with a confirmed booking on that driver's ride (and admin).
- `driver_bank`: SELECT only for the driver, admin, or a user holding a `confirmed` booking on a ride of that driver that accepts transfer.
- `routes`, `stops`, `route_fares`: SELECT active ones to anon and authenticated; writes admin RPCs only. `events`: SELECT published to everyone; drafts admin only.
- `rides`: via `public_departures` view (scheduled, future, active route, approved driver, remaining seats computed). `bookings`: rider reads own; driver reads those on own rides; admin all. `waiting_posts`: owner reads own; everyone else only `stop_demand`.
- `points_ledger`, `commissions`, `ratings`, `disputes`, `trip_events`, `referrals`, `notifications`: read-own where relevant, no client writes; `notifications` allows owner to mark `read_at` via RPC.
- Append-only triggers on `points_ledger`, `trip_events`, `audit_log`, `commissions` financial columns.
- All RPCs: `SECURITY DEFINER`, `SET search_path = public`, input validated, per-user rate limits via `rate_limits`.

**Platform:** HTTPS only, strict CSP and headers (`default-src 'self'`; `connect-src` Supabase project; `img-src` self, data:, OSM tiles, Supabase Storage; `script-src` self (+ Turnstile); `frame-ancestors 'none'`; Paystack via full-page redirect so no Paystack script is needed), Cloudflare Turnstile captcha on sign-in, secrets only in Supabase secrets/host env (service role key never in the client or repo), no PII in logs, `npm audit` + secret scanning in CI, least-privilege, weekly `pg_dump` to private storage via GitHub Action (free tier has no managed backups). Be honest in README: this makes abuse *hard*, not "impossible"; list residual risks.

---

## 22. INTEGRATIONS AND API KEYS (step by step; all free to start)

| # | Service | What you do | Gives you |
|---|---|---|---|
| 1 | **Supabase** (supabase.com, free plan) | Create project, region closest to Nigeria that is offered. Enable extensions `pg_cron`, `pgcrypto`. Auth → Providers → Google ON. Auth → Settings → enable captcha (Turnstile). Create Storage bucket `event-banners`. | `SUPABASE_URL`, `SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY` (server only) |
| 2 | **Google Cloud Console** | APIs & Services → OAuth consent screen (External) → Credentials → OAuth client (Web). Authorized redirect URI: `https://<project-ref>.supabase.co/auth/v1/callback`. Authorized JS origins: your site URLs and `http://localhost:5173`. Paste Client ID and Secret into Supabase Google provider. | Google sign-in |
| 3 | **Paystack** (paystack.com) | Create account, use **Test mode** keys first. Settings → API Keys: copy test secret key. Settings → Webhooks: URL `https://<project-ref>.supabase.co/functions/v1/paystack-webhook`. Set callback URL to `https://<site>/driver/fees`. Complete business verification later to go live. | `PAYSTACK_SECRET_KEY` (secret), webhook |
| 4 | **Cloudflare Turnstile** | Add site, get site key + secret. Site key to the client, secret to Supabase Auth captcha. | Bot protection |
| 5 | **Cloudflare Pages or Vercel** | Connect the GitHub repo, build `npm run build`, output `dist`. Add env vars below. | Hosting + HTTPS |
| 6 | **GitHub** | Repo + Actions (CI: typecheck, lint, unit tests, RLS tests, audit; weekly backup job). | CI |
| 7 | **OpenStreetMap** | No key. Keep the attribution on every map. | Map tiles |
| 8 | *Optional, flag OFF* VAPID keys | `npx web-push generate-vapid-keys` for web push. | Push notifications |

**Env vars (`.env.example` must list all with comments):**
```
# client (public)
VITE_SUPABASE_URL=  VITE_SUPABASE_ANON_KEY=  VITE_TURNSTILE_SITE_KEY=  VITE_APP_URL=
# edge function secrets (never in client): set with `supabase secrets set`
PAYSTACK_SECRET_KEY=  SUPABASE_SERVICE_ROLE_KEY=  APP_URL=
ADMIN_EMAILS=francisetham01@gmail.com   # add co-founder later, comma-separated; synced into admin_allowlist
VAPID_PUBLIC_KEY=  VAPID_PRIVATE_KEY=   # optional
```
Free-tier notes for the README: Supabase free projects can pause after a period of inactivity (wake before demos); Paystack live mode needs business verification; free tier limits on database size, storage, and edge invocations are fine for the pilot but add a usage check to the launch checklist.

---

## 23. DELIVERY PHASES (verify each; commit per phase)
1. **Foundations:** scaffold, design tokens + `DESIGN.md`, brand assets and favicon set, Supabase init, Google sign-in, `ensure_profile`, admin allowlist trigger and `is_admin()`, RLS v1, local-stack test harness.
2. **Routes and admin config:** migrations, seed 4 routes (terminals only), admin route/stop/fare UI (drag to reorder, coordinates).
3. **Offer a ride (driver):** onboarding, approval, offer ride, toggles, bank details, request handling, lock.
4. **Book a ride (rider):** routes, pickup/drop, departure list with live fare split, request, ticket with plate and safety popups.
5. **Backend core:** booking/cancel/lifecycle RPCs, `trip_events`, hold expiry cron, **concurrency test**, fare fixtures test.
6. **Money:** commission creation (5%), Paystack init + webhook (signature, verify, idempotency), owed/paid UI, enforcement, Money tab.
7. **Live icons:** waiting posts, `stop_demand`, map markers, Realtime, notifications.
8. **Events:** schema, admin wizard, event routes, home carousel, event page, end/cancel handling, broadcast.
9. **Points, referrals, profile Points box, avatar system, popups.**
10. **Squad teaser and disputes (backend + admin).**
11. **Remaining admin tabs** (Pulse, Trips, Drivers, Points, Issues, Admins/Config).
12. **Landing page**, responsive audit (320, 375, 768, 1024, 1440, 2560), a11y and Lighthouse pass.
13. **Hardening:** CSP/headers, captcha, RLS review, backups workflow, README with setup, env vars, deploy, tests, residual risks, launch checklist.

---

## 24. ACCEPTANCE CRITERIA
- A user can sign in with Google, offer a ride (approved driver) or book one, see the live fare split, and complete a trip.
- **By automated test**, a client cannot: change their role, points, seat counts, payment confirmation, fare, or driver status; read another user's booking; read bank details without a confirmed transfer booking on that driver; read `admin_allowlist`; write any table directly.
- Concurrent booking: 10 parallel requests for the last seat yield exactly 1 success.
- Fare vectors pass identically in SQL and TypeScript.
- Commission is created at **5%** (from config) on completion, and becomes `paid` only after a verified Paystack webhook; replaying the same webhook changes nothing.
- A driver over the owed threshold cannot offer a ride until settled.
- Waiting-rider icons show only real active posts and expose no rider ids or contact data.
- An admin can create, publish, and end an event; drivers can offer rides on its routes and riders can board; ending the event cancels open rides cleanly; non-live events are not bookable.
- Only allowlisted emails reach `/admin`.
- Fully usable at 320px and at 2560px; Lighthouse performance and a11y pass targets in section 19.
- `README.md` covers setup, env vars, deploy, tests, Assumptions, launch checklist, and residual risks. `.env.example` complete. `DESIGN.md` written.

---

## APPENDIX A: `public/avatars/sitidrive-avatars-sprite.svg` (create this file with exactly this content)
```svg
<svg xmlns="http://www.w3.org/2000/svg" style="display:none">
<symbol id="sd-avatar-01-rider-helmet" viewBox="0 0 96 96"><clipPath id="clip01"><circle cx="48" cy="48" r="48"/></clipPath><g clip-path="url(#clip01)"><rect width="96" height="96" fill="#CCFBF1"/><path d="M16 96 C16 78 31 71 48 71 C65 71 80 78 80 96 Z" fill="#1F2937"/><rect x="41" y="60" width="14" height="16" rx="5" fill="#5A3825"/><ellipse cx="48" cy="48" rx="17" ry="20" fill="#5A3825"/><circle cx="41" cy="48" r="1.9" fill="#1F2937"/><circle cx="55" cy="48" r="1.9" fill="#1F2937"/><path d="M42 58 Q48 63 54 58" fill="none" stroke="#1F2937" stroke-width="1.9" stroke-linecap="round"/><path d="M29 47 C26 20 70 20 67 47 C60 41 36 41 29 47 Z" fill="#F97316"/><path d="M48 22 V38" stroke="#fff" stroke-width="3" stroke-linecap="round"/><rect x="31" y="41" width="34" height="3" rx="1.5" fill="#1F2937" opacity=".35"/></g></symbol>
<symbol id="sd-avatar-02-gele-headwrap" viewBox="0 0 96 96"><clipPath id="clip02"><circle cx="48" cy="48" r="48"/></clipPath><g clip-path="url(#clip02)"><rect width="96" height="96" fill="#CCFBF1"/><path d="M16 96 C16 78 31 71 48 71 C65 71 80 78 80 96 Z" fill="#14B8A6"/><rect x="41" y="60" width="14" height="16" rx="5" fill="#7B4B2A"/><ellipse cx="48" cy="48" rx="17" ry="20" fill="#7B4B2A"/><circle cx="41" cy="48" r="1.9" fill="#1F2937"/><circle cx="55" cy="48" r="1.9" fill="#1F2937"/><path d="M42 58 Q48 63 54 58" fill="none" stroke="#1F2937" stroke-width="1.9" stroke-linecap="round"/><path d="M27 44 C21 24 40 13 55 18 C71 21 75 37 67 45 C60 38 35 38 27 44 Z" fill="#EC4899"/><path d="M55 18 C57 7 74 11 66 23 Z" fill="#F87171"/><path d="M33 36 C42 30 55 30 63 36" stroke="#fff" stroke-width="1.5" fill="none" opacity=".6"/><circle cx="31" cy="57" r="2.2" fill="#F97316"/><circle cx="65" cy="57" r="2.2" fill="#F97316"/></g></symbol>
<symbol id="sd-avatar-03-fade-glasses" viewBox="0 0 96 96"><clipPath id="clip03"><circle cx="48" cy="48" r="48"/></clipPath><g clip-path="url(#clip03)"><rect width="96" height="96" fill="#FFEDD5"/><path d="M16 96 C16 78 31 71 48 71 C65 71 80 78 80 96 Z" fill="#1F2937"/><rect x="41" y="60" width="14" height="16" rx="5" fill="#3B2219"/><ellipse cx="48" cy="48" rx="17" ry="20" fill="#3B2219"/><circle cx="41" cy="48" r="1.9" fill="#1F2937"/><circle cx="55" cy="48" r="1.9" fill="#1F2937"/><path d="M42 58 Q48 63 54 58" fill="none" stroke="#1F2937" stroke-width="1.9" stroke-linecap="round"/><path d="M30 45 C28 24 68 24 66 45 C63 37 33 37 30 45 Z" fill="#111827"/><g fill="none" stroke="#F97316" stroke-width="1.8"><circle cx="40.5" cy="48" r="6"/><circle cx="55.5" cy="48" r="6"/><path d="M46.5 48 H49.5"/></g></g></symbol>
<symbol id="sd-avatar-04-afro-puff" viewBox="0 0 96 96"><clipPath id="clip04"><circle cx="48" cy="48" r="48"/></clipPath><g clip-path="url(#clip04)"><rect width="96" height="96" fill="#CCFBF1"/><ellipse cx="48" cy="38" rx="25" ry="25" fill="#111827"/><path d="M16 96 C16 78 31 71 48 71 C65 71 80 78 80 96 Z" fill="#EC4899"/><rect x="41" y="60" width="14" height="16" rx="5" fill="#8D5524"/><ellipse cx="48" cy="48" rx="17" ry="20" fill="#8D5524"/><circle cx="41" cy="48" r="1.9" fill="#1F2937"/><circle cx="55" cy="48" r="1.9" fill="#1F2937"/><path d="M42 58 Q48 63 54 58" fill="none" stroke="#1F2937" stroke-width="1.9" stroke-linecap="round"/><path d="M31 44 C32 30 64 30 65 44 C60 38 36 38 31 44 Z" fill="#111827"/><circle cx="31" cy="57" r="2.2" fill="#F97316"/><circle cx="65" cy="57" r="2.2" fill="#F97316"/></g></symbol>
<symbol id="sd-avatar-05-hijab" viewBox="0 0 96 96"><clipPath id="clip05"><circle cx="48" cy="48" r="48"/></clipPath><g clip-path="url(#clip05)"><rect width="96" height="96" fill="#FCE7F3"/><path d="M18 96 C16 68 24 54 24 44 C24 26 36 20 48 20 C60 20 72 26 72 44 C72 54 80 68 78 96 Z" fill="#14B8A6"/><path d="M16 96 C16 78 31 71 48 71 C65 71 80 78 80 96 Z" fill="#1F2937"/><rect x="41" y="60" width="14" height="16" rx="5" fill="#A9693A"/><ellipse cx="48" cy="50" rx="15" ry="18" fill="#A9693A"/><circle cx="41" cy="50" r="1.9" fill="#1F2937"/><circle cx="55" cy="50" r="1.9" fill="#1F2937"/><path d="M42 60 Q48 65 54 60" fill="none" stroke="#1F2937" stroke-width="1.9" stroke-linecap="round"/><path d="M30 52 C28 30 68 30 66 52 C66 66 58 72 48 72 C38 72 30 66 30 52 Z" fill="none" stroke="#14B8A6" stroke-width="5"/><path d="M30 72 C36 80 60 80 66 72" fill="none" stroke="#EC4899" stroke-width="3" stroke-linecap="round"/></g></symbol>
<symbol id="sd-avatar-06-locs" viewBox="0 0 96 96"><clipPath id="clip06"><circle cx="48" cy="48" r="48"/></clipPath><g clip-path="url(#clip06)"><rect width="96" height="96" fill="#CCFBF1"/><path d="M28 38 V66" stroke="#111827" stroke-width="4.5" stroke-linecap="round"/><path d="M33 38 V70" stroke="#111827" stroke-width="4.5" stroke-linecap="round"/><path d="M63 38 V70" stroke="#111827" stroke-width="4.5" stroke-linecap="round"/><path d="M68 38 V66" stroke="#111827" stroke-width="4.5" stroke-linecap="round"/><path d="M16 96 C16 78 31 71 48 71 C65 71 80 78 80 96 Z" fill="#F87171"/><rect x="41" y="60" width="14" height="16" rx="5" fill="#5A3825"/><ellipse cx="48" cy="48" rx="17" ry="20" fill="#5A3825"/><circle cx="41" cy="48" r="1.9" fill="#1F2937"/><circle cx="55" cy="48" r="1.9" fill="#1F2937"/><path d="M42 58 Q48 63 54 58" fill="none" stroke="#1F2937" stroke-width="1.9" stroke-linecap="round"/><path d="M29 45 C26 22 70 22 67 45 C60 36 36 36 29 45 Z" fill="#111827"/></g></symbol>
<symbol id="sd-avatar-07-braids" viewBox="0 0 96 96"><clipPath id="clip07"><circle cx="48" cy="48" r="48"/></clipPath><g clip-path="url(#clip07)"><rect width="96" height="96" fill="#FCE7F3"/><path d="M29 40 V70" stroke="#111827" stroke-width="3.5" stroke-linecap="round"/><circle cx="29" cy="73" r="2.4" fill="#14B8A6"/><path d="M34 40 V74" stroke="#111827" stroke-width="3.5" stroke-linecap="round"/><circle cx="34" cy="77" r="2.4" fill="#14B8A6"/><path d="M62 40 V74" stroke="#111827" stroke-width="3.5" stroke-linecap="round"/><circle cx="62" cy="77" r="2.4" fill="#14B8A6"/><path d="M67 40 V70" stroke="#111827" stroke-width="3.5" stroke-linecap="round"/><circle cx="67" cy="73" r="2.4" fill="#14B8A6"/><path d="M16 96 C16 78 31 71 48 71 C65 71 80 78 80 96 Z" fill="#F97316"/><rect x="41" y="60" width="14" height="16" rx="5" fill="#7B4B2A"/><ellipse cx="48" cy="48" rx="17" ry="20" fill="#7B4B2A"/><circle cx="41" cy="48" r="1.9" fill="#1F2937"/><circle cx="55" cy="48" r="1.9" fill="#1F2937"/><path d="M42 58 Q48 63 54 58" fill="none" stroke="#1F2937" stroke-width="1.9" stroke-linecap="round"/><path d="M30 45 C27 22 69 22 66 45 C60 37 36 37 30 45 Z" fill="#111827"/><path d="M48 24 V37" stroke="#7B4B2A" stroke-width="1.6"/></g></symbol>
<symbol id="sd-avatar-08-cap-beard" viewBox="0 0 96 96"><clipPath id="clip08"><circle cx="48" cy="48" r="48"/></clipPath><g clip-path="url(#clip08)"><rect width="96" height="96" fill="#CCFBF1"/><path d="M16 96 C16 78 31 71 48 71 C65 71 80 78 80 96 Z" fill="#1F2937"/><rect x="41" y="60" width="14" height="16" rx="5" fill="#A9693A"/><ellipse cx="48" cy="48" rx="17" ry="20" fill="#A9693A"/><circle cx="41" cy="48" r="1.9" fill="#1F2937"/><circle cx="55" cy="48" r="1.9" fill="#1F2937"/><path d="M42 58 Q48 63 54 58" fill="none" stroke="#1F2937" stroke-width="1.9" stroke-linecap="round"/><path d="M32 53 C34 72 62 72 64 53 C60 62 36 62 32 53 Z" fill="#111827"/><path d="M29 42 C29 22 67 22 67 42 Z" fill="#F87171"/><rect x="27" y="40" width="48" height="4.5" rx="2.2" fill="#EF4444"/></g></symbol>
<symbol id="sd-avatar-09-headphones" viewBox="0 0 96 96"><clipPath id="clip09"><circle cx="48" cy="48" r="48"/></clipPath><g clip-path="url(#clip09)"><rect width="96" height="96" fill="#FEE2E2"/><path d="M16 96 C16 78 31 71 48 71 C65 71 80 78 80 96 Z" fill="#EC4899"/><rect x="41" y="60" width="14" height="16" rx="5" fill="#C68642"/><ellipse cx="48" cy="48" rx="17" ry="20" fill="#C68642"/><circle cx="41" cy="48" r="1.9" fill="#1F2937"/><circle cx="55" cy="48" r="1.9" fill="#1F2937"/><path d="M42 58 Q48 63 54 58" fill="none" stroke="#1F2937" stroke-width="1.9" stroke-linecap="round"/><path d="M30 45 C29 28 67 28 66 45 C62 38 34 38 30 45 Z" fill="#111827"/><path d="M28 48 C26 14 70 14 68 48" fill="none" stroke="#1F2937" stroke-width="4" stroke-linecap="round"/><rect x="23" y="44" width="9" height="15" rx="4.5" fill="#F97316"/><rect x="64" y="44" width="9" height="15" rx="4.5" fill="#F97316"/></g></symbol>
<symbol id="sd-avatar-10-elder-glasses" viewBox="0 0 96 96"><clipPath id="clip10"><circle cx="48" cy="48" r="48"/></clipPath><g clip-path="url(#clip10)"><rect width="96" height="96" fill="#E5E7EB"/><path d="M16 96 C16 78 31 71 48 71 C65 71 80 78 80 96 Z" fill="#F97316"/><rect x="41" y="60" width="14" height="16" rx="5" fill="#5A3825"/><ellipse cx="48" cy="48" rx="17" ry="20" fill="#5A3825"/><circle cx="41" cy="48" r="1.9" fill="#1F2937"/><circle cx="55" cy="48" r="1.9" fill="#1F2937"/><path d="M42 58 Q48 63 54 58" fill="none" stroke="#1F2937" stroke-width="1.9" stroke-linecap="round"/><path d="M30 45 C28 26 68 26 66 45 C63 38 33 38 30 45 Z" fill="#9CA3AF"/><path d="M42 56 Q48 54 54 56" stroke="#9CA3AF" stroke-width="2.2" fill="none" stroke-linecap="round"/><g fill="none" stroke="#1F2937" stroke-width="1.8"><circle cx="40.5" cy="48" r="6"/><circle cx="55.5" cy="48" r="6"/><path d="M46.5 48 H49.5"/></g></g></symbol>
</svg>
```

## APPENDIX B: `public/avatars/tokens.json` (create with exactly this content)
```json
{
  "color": { "charcoal": "#1F2937", "orange": "#F97316", "teal": "#14B8A6", "magenta": "#EC4899", "mutedRed": "#F87171" },
  "skinTones": { "deep": "#3B2219", "rich": "#5A3825", "brown": "#7B4B2A", "warm": "#8D5524", "caramel": "#A9693A", "light": "#C68642" },
  "avatarBackgrounds": { "or": "#FFEDD5", "te": "#CCFBF1", "ma": "#FCE7F3", "re": "#FEE2E2", "ch": "#E5E7EB" },
  "avatarSizes": { "xs": 24, "sm": 32, "md": 40, "lg": 56, "xl": 96 },
  "avatars": [
    { "id": "sd-avatar-01-rider-helmet", "label": "Rider with helmet" },
    { "id": "sd-avatar-02-gele-headwrap", "label": "Woman with gele headwrap" },
    { "id": "sd-avatar-03-fade-glasses", "label": "Man with fade and glasses" },
    { "id": "sd-avatar-04-afro-puff", "label": "Woman with afro" },
    { "id": "sd-avatar-05-hijab", "label": "Woman in hijab" },
    { "id": "sd-avatar-06-locs", "label": "Man with locs" },
    { "id": "sd-avatar-07-braids", "label": "Woman with braids" },
    { "id": "sd-avatar-08-cap-beard", "label": "Man with cap and beard" },
    { "id": "sd-avatar-09-headphones", "label": "Commuter with headphones" },
    { "id": "sd-avatar-10-elder-glasses", "label": "Elder with grey hair and glasses" }
  ]
}
```
Note: the manifest in the avatar handoff pack also lists a `file` path per avatar; the sprite is the source of truth here, so the field is optional.
