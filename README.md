# OTA Mobility — Scheduled Commuter Bus Platform

Scheduled, subscription-based commuter bus service connecting Ota/Ogun residential and industrial hubs to Lagos employment corridors.

Replaces informal "fill-and-go" danfo/keke with a published, enforced timetable, guaranteed seating, fixed pricing, and verified safety.

> Schedule is a contract, not a suggestion. Buses depart at T-0 regardless of load.

## Core Principles

1. **Schedule is a contract** — buses depart at T-0 regardless of load
2. **Seat is guaranteed** — no boarding disputes
3. **Price is fixed** — no surge, ever
4. **Access is inclusive** — app, WhatsApp, USSD, in-person kiosk (zero smartphone/data required)
5. **Safety is non-negotiable** — verified drivers, GPS, insurance, SOS
6. **Local ops is the real product** — software supports Junction Coordinators, doesn't replace them

## User Groups (2 only)

| Group | Subtype | Channel |
|-------|---------|---------|
| **RIDER** | All commuters (corporate, shift, feature-phone — channel preference only) | Native app / WhatsApp bot / USSD / kiosk walk-up |
| **ADMIN** | `driver` | Driver view in Admin app |
| **ADMIN** | `coordinator` | Kiosk/agent view in Admin app (POS-style) |
| **ADMIN** | `dispatcher` | Control Tower web dashboard |
| **ADMIN** | `employer-admin` (Phase 2) | Employer portal |
| **ADMIN** | `super-admin` | Control Tower + admin settings |

Rules:
- `User.user_group` is `RIDER` or `ADMIN` only — drivers are `ADMIN (admin_role=driver)`, not a third group
- `User.admin_role` is NULL for riders, required for admins
- Single Admin app surfaces views by `admin_role` — no separate auth systems
- Phone-number + OTP auth, RBAC enforced server-side on `user_group + admin_role`

## Modules

- **A. Rider Booking & Ticketing** — browse corridors/slots, reserve seat, pay (wallet/card/transfer/cash code), QR + 6-digit SMS fallback, rollover (≥30min notice), live ETA
- **B. WhatsApp Bot** — menu booking, ticket image/PDF, delay alerts, human handoff
- **C. USSD Gateway** — full booking + wallet on feature phone, SMS boarding code
- **D. Kiosk / Coordinator View (ADMIN)** — assisted booking, cash-to-wallet + SMS receipt, QR/SMS validation, float + daily settlement log
- **E. Driver View (ADMIN)** — daily assignment, live manifest, 1-tap incident/SOS, GPS beacon
- **F. Control Tower (ADMIN dispatcher)** — live map, T-15 staging / T-5 boarding / T-2 gate close / T-0 departure, 1-click standby dispatch, delay broadcast
- **G. Wallet & Payments** — closed-loop wallet, append-only double-entry ledger
- **H. Passes** — Weekly (10 trips, 10-12% off), Monthly (40 trips, 18-20% off), auto-renew
- **I. Waitlist & Rollover** — T-5 auto-release, rollover credit on qualifying cancel
- **J. Notifications** — fan-out push/WhatsApp/SMS
- **K. Safety** — SOS → Control Tower + GPS log, driver ratings + incident log

## Tech Stack

- **Mobile (rider + admin):** React Native
- **Web (Control Tower, booking fallback):** Next.js + TypeScript
- **Backend:** Node.js + NestJS (or Django — pick one, don't mix)
- **DB:** PostgreSQL + PostGIS
- **Cache / seat-lock:** Redis (short-TTL distributed locks)
- **Async:** Kafka or AWS SQS/SNS
- **GPS/ETA:** MQTT (EMQX) → ingestion → WebSocket (Socket.io), Redis hot + PostGIS history
- **USSD/SMS:** Africa's Talking or Termii
- **WhatsApp:** WhatsApp Business Cloud API (official)
- **Payments:** Paystack or Flutterwave (tokenized, never touch raw card data)
- **Maps:** Mapbox or Google Maps
- **Hosting:** AWS/GCP (eu-west or NG edge), Terraform IaC, dev/staging/prod

## Roadmap

| Phase | Timeline | Scope |
|-------|----------|-------|
| **Phase 0 — Manual Validation** | Weeks 0-8 | No prod code. 50 interviews + 2-week concierge pilot (2 buses, WhatsApp, cash/transfer, paper manifests) |
| **Phase 1 — Pilot Launch (MVP)** | Months 3-6 | App + WhatsApp + USSD, Driver/Admin views, Control Tower, wallet/ledger, passes, notifications. 3 corridors, 6-10 vehicles |
| **Phase 2 — Network Scaling** | Months 6-12 | Auto dispatch, standby logic, Employer/B2B portal, 8-10 corridors |
| **Phase 3 — Regional Network** | Months 12-24 | Demand forecasting, fleet rebalancing, enterprise API, CNG/EV telemetry |

MVP is done when: book via all 3 channels → validated boarding in 5-min window → T-0 departure visible live → >15min delay triggers 1-click standby + auto-notify → ledger correct, no double-book/charge → KPIs auto-captured.

## Success Metrics

- **Rider:** wait <10min, on-time ≥90%, seat fulfillment 100%, NPS >40, pass renewal ≥70%
- **Ops:** load ≥85%, adherence ≥95%, cancel <2%, boarding <5min, uptime ≥98%
- **Financial:** revenue/bus/day, margin/corridor, CAC, LTV, auto-renew conversion
- **Safety:** 0 incidents/100k km, <1 complaints/1000 trips, driver rating ≥4.5/5

## Docs

- `PRD.md` — full PRD + TRD (source of truth)
- `CREATIVE IDEATION_ Scheduled Commuter Bus Service in Ota.docx` — founder ideation (problems, wants, junctions, slots)

### Key Corridors

Pickups: Sango Ota, Joju, Toll Gate, Iyana Iyesi/Bells Drive, Owode Ota
Drop-offs: Agbara Industrial Estate, Idiroko Road factories, Lagos (Ikeja, Oshodi, VI, Lekki, Berger)
Slots AM: 5:00, 5:30, 6:00, 6:30, 7:00, 7:30, 8:00, 9:00 — PM: 4:30, 5:30, 6:30, 7:30, 8:30

## Getting Started (code — Phase 1)

Repo is currently docs-only. When Phase 1 starts:

```bash
# backend (NestJS example)
# npm install && npm run start:dev

# web (Control Tower)
# npm install && npm run dev
```

Env needed: `DATABASE_URL`, `REDIS_URL`, `PAYSTACK_SECRET`, `WHATSAPP_TOKEN`, `AFRICASTALKING_KEY`, `MQTT_BROKER_URL`.

---
Built for Ota ↔ Lagos commuters. Schedule is a contract.
