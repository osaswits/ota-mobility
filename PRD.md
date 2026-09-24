# OTA MOBILITY — Scheduled Commuter Bus Platform

## Full-Stack Product & Technical Requirements Document (Build Prompt)

**Prepared as an execution-ready prompt/spec for a development team or AI coding agent (e.g. Base44, Bolt, Cursor, an in-house eng team).** **Source material:** `CREATIVE_IDEATION: Scheduled Commuter Bus Service in Ota` \+ `Strategic Architecture and Product Execution Framework for a Scheduled Mobility Guarantee in Ota` (both supplied by the founder).

---

## 0\. HOW TO USE THIS DOCUMENT (Builder Instructions)

> You are a senior full-stack engineering team with deep experience in transport-tech, fleet telemetry, and fintech-adjacent ledger systems, building for a low-bandwidth, price-sensitive, partially feature-phone West African commuter market (Ota, Ogun State ↔ Lagos, Nigeria). You are NOT building a generic ride-hailing clone. You are building a **scheduled mobility guarantee**: fixed departures, guaranteed seats, fixed pricing — the schedule is a contract, not a suggestion. Every technical decision should be evaluated against: *"Does this help a bus leave a junction at exactly the published time, every time, even in bad network conditions and with cash-paying customers?"* Build in phases exactly as scoped in Section 6\. Do not gold-plate Phase 0/1 with Phase 3 infrastructure (predictive ML, EV fleet APIs, enterprise integrations) — those are explicitly deferred.

---

## 1\. PRODUCT REQUIREMENTS (PRD)

### 1.1 Vision

A scheduled, subscription-based commuter bus service connecting Ota/Ogun residential and industrial hubs to Lagos employment corridors, replacing the informal "fill-and-go" danfo/keke model with a published, enforced timetable, guaranteed seating, fixed pricing, and verified safety standards.

### 1.2 Core Product Principles (non-negotiable design constraints)

1. **Schedule is a contract** — buses depart at T-0 regardless of load.  
2. **Seat is guaranteed** — no boarding disputes.  
3. **Price is fixed** — no surge, ever.  
4. **Access is inclusive** — must work on app, WhatsApp, USSD, and in-person at a kiosk with zero smartphone/data requirement.  
5. **Safety is non-negotiable** — verified drivers, GPS, insurance, SOS.  
6. **Local ops is the real product** — software supports Junction Coordinators and staging bays; it does not replace them.

### 1.3 User Groups & Personas (simplified: 2 groups only)

There are exactly two user groups in the system: `RIDER` and `ADMIN`. Drivers are NOT a separate user group — they are a subtype of `ADMIN`.

| Group | Persona / Subtype | Primary Channel | Core Need |
| :---- | :---- | :---- | :---- |
| **RIDER** | All commuters (corporate, factory/shift, price-sensitive/feature-phone — same identity, channel preference only) | Native app / WhatsApp bot / USSD / walk-up at kiosk | Seat reservation, live ETA, subscription pass, cash-to-wallet |
| **ADMIN** — `admin_role=driver` | Driver | Driver view in Admin app | Route/schedule assignment, manifest, incident reporting |
| **ADMIN** — `admin_role=coordinator` | Junction Coordinator | Kiosk/agent view in Admin app (POS-style) | Manifest, boarding validation, cash reconciliation |
| **ADMIN** — `admin_role=dispatcher` | Ops Dispatcher | Control Tower web dashboard | Fleet visibility, delay management, standby dispatch |
| **ADMIN** — `admin_role=employer-admin` (Phase 2) | Employer HR admin | Employer portal | Bulk pass purchase, employee transport benefit management |
| **ADMIN** — `admin_role=super-admin` | Platform owner/ops lead | Control Tower + admin settings | User/role management, ledger audit, corridor/fare config |

Rules:
- `User.user_group` is either `RIDER` or `ADMIN` — no third top-level type.
- `User.admin_role` is NULL for riders; required for admins: `driver | coordinator | dispatcher | employer-admin | super-admin`.
- A single Admin app surfaces different views by `admin_role` (driver view vs. kiosk view) — do not build separate auth systems.

### 1.4 Functional Requirements by Module

**A. Rider Booking & Ticketing**

- Browse corridors/schedules by origin junction, destination, and time slot.  
- Reserve a specific seat on a specific scheduled trip.  
- Pay via wallet, card (tokenized), bank transfer, or cash top-up code.  
- Receive a QR code \+ fallback 6-digit SMS code as the ticket.  
- Reschedule/cancel per rollover policy (≥30 min notice → rollover credit, minus rescheduling fee).  
- View live GPS ETA of assigned bus to the junction.

**B. WhatsApp Conversational Bot**

- Menu-driven booking (corridor → time slot → seat count → payment).  
- Delivers ticket as image/PDF with QR \+ code.  
- Sends automated delay/standby-dispatch alerts.  
- Basic support handoff to human agent.

**C. USSD Gateway**

- Full booking \+ wallet payment flow on a feature phone, zero data.  
- SMS delivery of 6-digit boarding code.

**D. Junction Kiosk / Coordinator View (ADMIN, admin_role=coordinator)**

- Agent-assisted booking for walk-ups.  
- Cash-to-wallet top-up with instant SMS receipt.  
- QR/SMS code validation (boarding).  
- Digital float tracking with daily settlement log (anti-leakage control).

**E. Driver View (ADMIN, admin_role=driver)**

- Daily assignment: route, departure time, staging bay.  
- Live passenger manifest (who's checked in vs. no-show).  
- One-tap incident/SOS reporting.  
- GPS beacon (background location share while on shift).

**F. Control Tower (ADMIN, admin_role=dispatcher / super-admin)**

- Real-time map of all active \+ standby vehicles.  
- Schedule adherence monitoring (T-15 staging, T-5 boarding, T-2 gate close, T-0 departure).  
- One-click standby-bus dispatch when a primary vehicle is delayed \>15 min.  
- Delay-alert broadcast trigger (fans out to WhatsApp/SMS/push automatically).

**G. Wallet & Payments**

- Closed-loop digital wallet per user.  
- Funding: card, bank transfer, USSD payment code, cash-at-kiosk.  
- Full transaction ledger (append-only, auditable).

**H. Subscriptions/Passes**

- Weekly pass (10 trips, 10–12% discount), auto-renewing.  
- Monthly pass (40 trips, 18–20% discount), auto-renewing.  
- B2B corporate pass allocations (Phase 2).

**I. Waitlist & Rollover Engine**

- Auto-release unclaimed reserved seats to waitlist at T-5.  
- Auto-issue rollover credit for cancellations ≥30 min out (minus fee).

**J. Notifications**

- Channel-agnostic fan-out (push / WhatsApp / SMS) for: booking confirmation, T-15 boarding reminder, delay/standby alerts, pass renewal, low-wallet-balance.

**K. Safety**

- In-app/in-bot SOS button → alerts Control Tower \+ logs GPS position \+ timestamp.  
- Driver rating \+ safety-incident log feeding the KPI dashboard.

### 1.5 Explicit Non-Goals for MVP (Phase 0/1)

- No dynamic/surge pricing engine (contradicts the core value prop — must never be built).  
- No employer/B2B portal (Phase 2).  
- No predictive demand forecasting or ML-based dispatch (Phase 3).  
- No EV/CNG fleet telemetry integration (Phase 3).  
- No native driver-owned fleet management (asset-light — buses are leased, not owned).

### 1.6 Success Metrics (must be instrumented from day 1, not bolted on later)

- **Rider:** junction wait time (\<10 min), on-time departure rate (≥90%), seat-guarantee fulfillment (100%), NPS (\>40), pass renewal rate (≥70%).  
- **Ops:** load factor (≥85%), schedule adherence (≥95%), cancellation rate (\<2%), boarding duration (\<5 min), vehicle uptime (≥98%).  
- **Financial:** revenue/bus/day, contribution margin/corridor, CAC, LTV, pass auto-renewal conversion.  
- **Safety:** incidents/100k km (target 0), complaints/1,000 trips (\<1), driver safety rating (≥4.5/5).

---

## 2\. TECHNICAL REQUIREMENTS (TRD)

### 2.1 High-Level Architecture

┌─────────────────────────── OMNICHANNEL ACCESS LAYER ───────────────────────────┐  
│  Rider App (iOS/Android)  |  WhatsApp Bot  |  USSD Gateway  |  Admin App (driver + kiosk views) │  
└──────────────────────────────────┬──────────────────────────────────────────────┘  
                                    ▼  
                         API GATEWAY (auth, rate limit, routing)  
                                    ▼  
        ┌───────────────────────────────────────────────────────────┐  
        │                    CORE SERVICE LAYER                     │  
        │  Identity/Auth · Booking · Ticketing · Wallet/Ledger       │  
        │  Subscription/Pass · Waitlist/Rollover · Notification      │  
        │  Dispatch & Telemetry · Driver Ops · Employer (Ph.2)       │  
        └───────────────────────────────────────────────────────────┘  
                                    ▼  
        ┌────────────────────┬─────────────────┬─────────────────┐  
        │ PostgreSQL+PostGIS │ Redis (cache/    │ Kafka/SQS        │  
        │ (system of record) │ session/seat-lock)│ (async events)  │  
        └────────────────────┴─────────────────┴─────────────────┘  
                                    ▼  
        THIRD-PARTY INTEGRATIONS: Payments · SMS/USSD · WhatsApp Cloud API ·  
        Maps/GPS ingestion (MQTT) · Push (FCM/APNs)  
                                    ▼  
                     ADMIN APP VIEWS (driver/kiosk)  \+  CONTROL TOWER WEB DASHBOARD

### 2.2 Recommended Stack (with rationale — swap only for a documented reason)

- **Mobile (rider app + admin app with driver/kiosk role views):** React Native (single codebase, fast iteration for a pilot; revisit native if GPS-battery performance becomes an issue at scale).  
- **Web (Control Tower, employer portal, booking web fallback):** Next.js (React) \+ TypeScript.  
- **Backend:** Node.js \+ NestJS (TypeScript throughout the stack lowers context-switching cost for a small team) OR Django if the team is stronger in Python — pick one, do not mix.  
- **Database:** PostgreSQL with PostGIS extension (geo-queries for junctions/corridors, ETA calc).  
- **Cache / seat-locking:** Redis (critical for preventing double-booking of the same seat — use short-TTL distributed locks during checkout).  
- **Async messaging:** Kafka or AWS SQS/SNS for event-driven flows (booking confirmed → trigger notification → trigger ledger entry).  
- **Real-time GPS/ETA:** MQTT broker (e.g. EMQX) ingesting device telemetry → ingestion service → WebSocket (Socket.io) push to Control Tower and rider app.  
- **USSD/SMS:** Africa's Talking or Termii (Nigeria-proven USSD \+ SMS gateways).  
- **WhatsApp:** WhatsApp Business Cloud API (official Meta API, not an unofficial wrapper — needed for reliability and ToS compliance).  
- **Payments:** Paystack or Flutterwave (card/bank transfer, Nigeria-native, handles PCI compliance so you never touch raw card data).  
- **Maps:** Mapbox or Google Maps Platform (evaluate cost at scale — Mapbox is typically cheaper for high-volume tile/ETA usage).  
- **Hosting:** AWS or GCP, region closest to Nigeria with acceptable latency (eu-west or a local Nigerian cloud provider for a CDN edge); plan for intermittent-connectivity resilience regardless of host.  
- **Observability:** structured logging (e.g. Pino) \+ Grafana/Prometheus or a hosted equivalent (Datadog) — non-negotiable given the schedule-adherence KPI depends on accurate timestamps.

### 2.3 Core Services (bounded contexts)

1. **Identity & Auth** — phone-number + OTP auth (not email-first; this market is phone-native). Exactly two user groups: `RIDER` and `ADMIN`. `ADMIN` has subtypes via `admin_role`: `driver | coordinator | dispatcher | employer-admin | super-admin`. RBAC checks `user_group + admin_role` on every endpoint. Riders use app/WhatsApp/USSD; admins use Admin app views + Control Tower.
2. **Booking & Reservation** — seat inventory per trip instance, distributed lock on seat-selection to prevent race conditions, holds expire after a short window if payment isn't completed.  
3. **Ticketing & Validation** — generates QR + 6-digit fallback code per booking; validation endpoint callable from admin views (kiosk/coordinator scan, driver view), or SMS-based check.  
4. **Wallet & Ledger** — append-only double-entry ledger (never mutate past transactions; corrections are new offsetting entries). This is the system of record for all money movement — treat it with the same rigor as a fintech ledger, because reconciliation disputes (cash handling, agent leakage — a named risk in the source doc) will happen.  
5. **Subscription/Pass** — pass definitions, auto-renewal billing, trip-decrement logic against a pass balance.  
6. **Waitlist & Rollover** — background job triggered at T-5 (release unclaimed seats) and on qualifying cancellations (issue rollover credit).  
7. **Dispatch & Fleet Telemetry** — GPS ingestion, geofence detection (bus entering/leaving staging bay), ETA computation, schedule-adherence tracking, standby-fleet auto-suggestion when a delay exceeds threshold.  
8. **Notification** — single fan-out service abstracting push/SMS/WhatsApp so business logic never talks to a channel API directly.  
9. **Driver Ops (Admin subtype view)** — assignment feed, manifest, incident reporting for `ADMIN admin_role=driver`. Shares auth, GPS pipeline, and notification plumbing with other admin views.  
10. **Analytics/KPI** — event pipeline feeding the four KPI categories in §1.6; build this from day one so Phase 0/1 pilot metrics are real, not guessed.  
11. **Employer/B2B Portal** (Phase 2 — stub the data model now, do not build the UI yet).

### 2.4 Core Data Model (entities — not exhaustive field lists)

`User (user_group: RIDER | ADMIN; admin_role: driver | coordinator | dispatcher | employer-admin | super-admin, NULL for riders)`, `Vehicle`, `Corridor`, `JunctionStagingBay`, `ScheduledTripTemplate` (recurring schedule definition), `TripInstance` (one concrete departure on one date), `Seat`, `Booking`, `Ticket`, `Wallet`, `LedgerEntry`, `Pass` / `PassSubscription`, `WaitlistEntry`, `IncidentReport`, `SafetyAlert`, `CashSettlementLog` (per coordinator, per day).

Key constraints to encode at the database level, not just app logic:

- A `Seat` can have at most one active `Booking` per `TripInstance` (unique constraint).  
- `LedgerEntry` rows are immutable once written.  
- `TripInstance.status` transitions are enforced as a state machine: `scheduled → staging → boarding → departed → completed | cancelled`.

### 2.5 API Surface (representative, group by service — expand during implementation)

POST   /auth/otp/request  
POST   /auth/otp/verify  
GET    /corridors  
GET    /corridors/:id/trips?date=\&window=  
POST   /bookings                      (seat hold \+ payment intent)  
POST   /bookings/:id/confirm  
POST   /bookings/:id/cancel  
GET    /tickets/:id/validate          (ADMIN only: coordinator/driver scan; enforced by user_group+admin_role)  
POST   /wallet/topup  
GET    /wallet/balance  
POST   /passes/subscribe  
GET    /passes/:id/status  
POST   /telemetry/gps                 (device → ingestion, high-frequency, auth via device token)  
GET    /control-tower/fleet/live  
POST   /control-tower/dispatch/standby  
POST   /incidents  
POST   /notifications/send            (internal only, called by other services)

### 2.6 Real-Time GPS/ETA Pipeline

Device (SIM-based GPS tracker or driver-phone GPS) → MQTT broker → ingestion service normalizes \+ geofences → writes latest position to Redis (hot) \+ PostGIS (historical) → ETA recalculated on each update → pushed via WebSocket to (a) rider app for their specific booked trip, (b) Control Tower for all active vehicles. Alert rule: if a `TripInstance` has not reached its staging-bay geofence by T-15, auto-flag to dispatcher for standby consideration.

### 2.7 Offline / Low-Bandwidth Design Requirements

- USSD and WhatsApp flows must never assume the rider has ever used the native app — they are first-class, independently complete booking paths, not "lite" versions.  
- Kiosk agent app should function on intermittent connectivity with local queuing \+ sync-on-reconnect (do not require an always-on connection to accept cash and issue a ticket).  
- SMS is the fallback of last resort for every notification type — never make push-only or WhatsApp-only the sole delivery path for a boarding code.

### 2.8 Security & Compliance

- Phone-based OTP auth; no passwords for riders. Admins (including drivers) authenticate the same way, with `user_group=ADMIN` + `admin_role` gating views/APIs.
- RBAC (user_group + admin_role) enforced server-side on every endpoint, not just hidden in the UI.  
- Payment card data never touches your servers — use Paystack/Flutterwave tokenization/hosted checkout.  
- NDPR (Nigeria Data Protection Regulation) compliance: data minimization, consent capture, right-to-erasure workflow, data residency consideration.  
- Encrypt data in transit (TLS everywhere) and at rest (DB-level encryption for wallet/ledger tables at minimum).  
- Audit log on every ledger mutation and every RBAC-privileged action (dispatcher standby-deploy, coordinator cash top-up).

### 2.9 Non-Functional Requirements

- **Availability:** ≥99.5% for booking/ticketing path (this is what riders touch daily); Control Tower can tolerate brief degradation before rider-facing services do.  
- **Latency:** seat-hold confirmation \<2s; GPS-to-map-update \<5s end-to-end.  
- **Scale target (pilot):** 300–500 completed trips/day across 6–10 vehicles → design for 10x that (3,000–5,000/day) before Phase 2 without re-architecture.  
- **Disaster recovery:** daily automated DB backups, point-in-time recovery for the ledger tables specifically.

### 2.10 DevOps

- Three environments minimum: dev, staging, production.  
- CI/CD with automated tests gating deploys (unit \+ integration, especially around seat-locking and ledger logic — these are the two places a bug becomes a money or trust problem).  
- Infra-as-code (Terraform) even for the pilot — the roadmap explicitly scales fleet/corridors fast, and hand-configured infra will not survive Phase 2\.

---

## 3\. PHASED BUILD ROADMAP (Engineering View)

| Phase | Timeline | What Gets Built |
| :---- | :---- | :---- |
| **Phase 0 — Manual Validation** | Weeks 0–8 | **No software beyond a spreadsheet \+ WhatsApp groups.** Do not write production code yet. Validate demand with 50 field interviews \+ a 2-week manual concierge pilot (2 buses, WhatsApp booking, cash/bank transfer, Junction Coordinators managing paper/spreadsheet manifests). |
| **Phase 1 — Pilot Launch** | Months 3–6 | Build: Native app \+ WhatsApp bot \+ USSD gateway (all three, not sequentially — they're equally first-class per §1.4), Driver app, manual/semi-automated Control Tower dashboard, wallet \+ ledger, pass subscriptions, automated notification fan-out. This is the MVP described in §1 and §2 above. Target: 3 corridors, 6–10 vehicles, 85% load factor, 70%+ pass renewal before moving on. |
| **Phase 2 — Network Scaling** | Months 6–12 | Automated dispatch tooling, dynamic standby-fleet allocation logic, Employer/B2B portal (build the deferred module now), expand to 8–10 corridors. |
| **Phase 3 — Regional Network** | Months 12–24 | Predictive demand forecasting, dynamic fleet rebalancing, enterprise ticketing API integrations, CNG/EV fleet telemetry support. |

**Build discipline:** do not let Phase 2/3 scope (ML forecasting, enterprise APIs, EV telemetry) leak into the Phase 1 codebase "for future-proofing." Stub the data model where cheap (e.g., a `Pass.employer_id` nullable field), but do not build the UI or the service logic early — it slows the pilot down and the pilot is what proves the business, not the architecture.

---

## 4\. DEFINITION OF DONE — MVP (Phase 1\)

The MVP is complete when, for a real commuter on a real corridor:

1. They can book a seat on a specific scheduled trip via **app, WhatsApp, or USSD** and get a valid QR/SMS ticket.  
2. A Junction Coordinator can validate that ticket and board them within the 5-minute boarding window.  
3. The bus departs at T-0 regardless of load, and the Control Tower dispatcher can see it live on a map.  
4. If the bus is delayed \>15 minutes pre-departure, a standby bus can be dispatched with one action and riders are auto-notified.  
5. The rider's wallet/ledger correctly reflects the fare deduction with no possibility of double-charge or double-booking of the same seat.  
6. All four KPI categories in §1.6 are being captured automatically, not manually tallied.

---

## 5\. OPEN QUESTIONS / ASSUMPTIONS REQUIRING FOUNDER VALIDATION

These are not resolved by the source documents and should not be assumed by the build team without founder sign-off:

- **Legal entity for fleet leasing/union relations** — who signs vehicle-lease and NURTW/RTEAN staging agreements: the founder's company directly, or a local operating partner?  
- **Payment settlement cadence with leased-fleet owners** — daily, weekly? This affects ledger design (does the platform hold float, or pass through instantly?).  
- **Insurance provider/mechanism** for "active passenger insurance" — third-party policy per trip, or a self-insured pool? This is a compliance and liability question, not just a technical one.  
- **GPS hardware vs. driver-phone GPS** for the pilot fleet — a dedicated SIM-based tracker is more reliable but adds per-vehicle hardware cost; driver-phone GPS is cheaper but fails if the driver's phone dies or the app is killed in the background.  
- **USSD/WhatsApp provider contracts** — Africa's Talking/Termii and Meta's WhatsApp Business API both have onboarding lead times and per-message costs that should be quoted before Phase 1 budgeting is finalized.

---

*Document assembled from the founder-supplied Product Strategy and Creative Ideation source material. Confidence in the product-requirements sections is high (directly sourced); confidence in specific technology picks (frameworks, hosting region, GPS hardware) is moderate — these are informed recommendations, not settled decisions, and should be validated against team skillset and vendor quotes before Phase 1 kickoff.*