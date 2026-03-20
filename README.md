# ZeroDown
**Parametric income protection for India's 10-minute delivery workforce**

*Guidewire DEVTrails 2026 · University Hackathon*

---

## Why this exists

A Zepto delivery partner in Hyderabad completes roughly 50 deliveries on a normal Tuesday. Then it rains — not just drizzle, but the kind of July downpour that turns Kondapur's service roads into shallow rivers. He parks his bike. He waits. Three hours later, he has lost ₹450 in income he was counting on to cover this week's household expenses.

Nobody calls him. No form exists for him to fill. The weather did not care about his earnings, and neither did any insurance product on the market.

ZeroDown closes that gap. When a verified external disruption hits his delivery zone, ZeroDown detects it automatically, confirms he was inactive during that window, and sends income replacement to his UPI account — before he even thinks to ask.

---

## The persona we chose, and why

**Grocery and Q-Commerce delivery partners — Zepto, Blinkit, Swiggy Instamart**

A food delivery partner completes 10–15 deliveries per day across a wide city radius. A Q-Commerce partner completes 40–80 deliveries per day, all within a dense 2–3 km micro-zone. When a disruption hits that micro-zone, the Q-Commerce worker loses far more income per hour than any other delivery category — their entire working territory is affected at once.

This also makes fraud detection actually reliable. Because Q-Commerce workers operate in fixed micro-zones anchored to a dark store, we can model their normal GPS behavior with precision. A food delivery partner could plausibly be anywhere in a city. A Blinkit partner should be within 3 km of one specific location. That zone-anchoring is the core of our anomaly detection approach.

**Reference persona — Ravi**

> Ravi, 24, Hyderabad. Delivers for Zepto from the Madhapur dark store.
> Earns ₹700–₹850 per day, 45–55 deliveries.
> Weekly income: ₹4,900–₹5,950. No savings buffer.
> Does not read insurance documents. Responds to WhatsApp messages in Telugu.

Every design decision passes through this filter: would Ravi understand it, trust it, and use it in under two minutes?

---

## Coverage scope

**We insure:** Lost income during verified external disruptions — hours in which a worker could not operate because of an event entirely outside their control.

**We do not insure:** Vehicle repairs, health or injury costs, accident medical bills, or any event the worker could have influenced.

This boundary is not a limitation — it is the product. Parametric insurance works because the trigger is objective. There is no ambiguity about whether it rained 15mm in the past hour. There is no investigation, no disputed claim. The event either happened or it did not.

---

## The problem in numbers

| Metric | Reality |
|---|---|
| Q-Commerce deliveries per day | 40–80 (vs 8–15 for food delivery) |
| Income lost per 3-hr flood disruption | ₹450–₹600 |
| Income lost to a full-day AQI shutdown | ₹700–₹850 |
| Monthly income at risk during monsoon | ~28% |
| Existing income protection products for gig workers | 0 |

---

## How ZeroDown works — Ravi's journey

```
┌──────────┬──────────┬──────────┬──────────┬──────────┐
│  STEP 1  │  STEP 2  │  STEP 3  │  STEP 4  │  STEP 5  │
│          │          │          │          │          │
│ Onboard  │  Policy  │Disruption│  Fraud   │  Payout  │
│ via OTP  │activated │ detected │  check   │ via UPI  │
│          │          │          │          │          │
│ 2 mins   │ ₹49–₹99  │Automatic │ <30 secs │ <5 mins  │
│ on phone │ per week │ 24/7     │ AI model │WhatsApp  │
└──────────┴──────────┴──────────┴──────────┴──────────┘
                              No claim form. Ever.
```

---

## Weekly premium model

Gig workers are paid weekly. Their expenses cycle weekly. ZeroDown matches that rhythm exactly.

### Base tiers

| Plan | Weekly premium | Max weekly payout | Disrupted hours covered |
|---|---|---|---|
| Basic | ₹49 | ₹1,500 | Up to 8 hours |
| Standard | ₹79 | ₹2,500 | Up to 14 hours |
| Full | ₹99 | ₹4,000 | Up to 20 hours |

### How the premium adjusts week to week

Every Sunday night, ZeroDown recalculates next week's premium using three signals:

**Zone risk score** — Each micro-zone has a 12-month disruption frequency score. A dark store in a low-lying area that floods twice per monsoon season carries a higher multiplier than one on elevated ground 2 km away. Updates quarterly.

**Forecast factor** — If the 7-day forecast shows a weather system approaching, premiums for affected zones rise modestly that week. Workers are notified in advance via WhatsApp.

**Loyalty modifier** — Workers with no prior fraudulent claims earn up to a 10% reduction over time. Honest use is rewarded without penalizing anyone for legitimate claims.

```
Final weekly premium =
  Base tier × Zone risk (0.80–1.30) × Forecast (0.90–1.20) × Loyalty (0.90–1.00)
```

**Example:** Ravi is on Standard (₹79 base). His zone is moderate-flood risk (×1.10). The forecast for July 15 shows heavy rain (×1.15). No fraud history (×1.00). His premium that week: ₹79 × 1.10 × 1.15 = **₹100**, sent Sunday evening.

---

## Parametric triggers

Five events most correlated with income loss for Hyderabad-based Q-Commerce workers. Different cities would surface different triggers.

| Trigger | Data source | Threshold | Payout type |
|---|---|---|---|
| Heavy rainfall | Open-Meteo + IMD advisory | > 15mm/hr in worker's zone | Per disrupted hour |
| Extreme heat | OpenWeatherMap | > 42°C sustained 3+ hours | Per disrupted hour |
| Severe AQI | CPCB real-time feed | Index > 300 (Hazardous) | Flat daily amount |
| Flash flood / waterlogging | State DMA API + GPS zone flag | Zone declared waterlogged | Per disrupted hour |
| Curfew or civic disruption | Government order API + news signal | Official restriction in delivery zone | Full shift amount |

**Payout formula:**

```
Payout = disrupted hours × worker's 4-week average hourly rate × 0.80
```

The 0.80 factor (80% income replacement) prevents any incentive to stay idle during marginal conditions while still providing meaningful coverage.

---

## AI and ML plan

### At onboarding — risk profiling

When Ravi registers, a background model runs silently using his delivery zone's disruption history, his platform tenure, and the seasonal risk profile for the current month. Output: a risk score (0–100) recommending a plan tier. Model: XGBoost gradient-boosted classifier. No personal creditworthiness or health data is used — only environmental exposure.

### Every Sunday — dynamic premium calculation

A time-series regression model ingests the 7-day weather forecast, current AQI trend, and any active government advisories. Output: the multiplier applied to next week's base premium. The model updates its weights from its own weekly prediction errors.

### At claim time — fraud detection (under 30 seconds)

Three independent signals feed an Isolation Forest anomaly detector:

**GPS zone consistency** — Was the worker's phone inside the declared disruption zone during the trigger window? A claim for a Madhapur flood with GPS showing the worker in Secunderabad is an immediate flag.

**Platform activity cross-reference** — Did the platform's delivery records show inactivity during the claimed window? Eight completed deliveries in the hour a worker claims weather inactivity is a logical contradiction.

**Claim frequency baseline** — We model normal claim frequency per zone per season. A worker claiming every week in a zone that historically triggers once per month is a statistical outlier.

Claims with fraud scores above 0.70 are held for a 2-hour manual review. All others pay automatically.

### For the insurer — weekly disruption forecast

A Prophet time-series model on the admin dashboard forecasts next week's likely disruption claims by zone, fed by historical trigger frequency and the upcoming weather outlook.

### Worker-facing — plain language claim explanations

If a claim is held or rejected, Ravi receives a WhatsApp message explaining why in plain language, in Telugu or Hindi. An LLM API call translates the system's internal fraud score reasoning into a two-sentence human explanation. This is the most important UX investment in the product — confusion about a rejected claim destroys trust faster than any other failure mode.

---

## Platform choice

**Progressive Web App (React) — not a native mobile app**

Q-Commerce workers use mid-range Android devices with limited storage. A PWA installs from a shared WhatsApp link, works offline for dashboard viewing, and sends push notifications — without competing for phone storage with the Zepto partner app. The key insight: the primary claim action is automatic. Ravi does not need to open the app to file a claim. The app is for checking coverage status, viewing payout history, and understanding next week's premium. That use pattern suits a PWA far better than a heavy native application.

---

## Tech stack

| Layer | Technology | Reason |
|---|---|---|
| Frontend | React + Tailwind CSS + Recharts | PWA-capable, fast to build, well-suited for dashboard charts |
| Backend | Python + FastAPI | Python-native ML serving, no language boundary for model endpoints |
| Database | MySQL + Redis | Persistent records + real-time trigger state |
| ML | Scikit-learn, XGBoost, Prophet | Best-fit tools for each ML task; all Python, all serializable |
| Weather | Open-Meteo (free) + IMD | Free tier sufficient for proof-of-concept, real data |
| AQI | CPCB public API | Official government source, real-time |
| Payments | Razorpay test mode | Full UPI simulation without real transactions |
| Notifications | Twilio trial | WhatsApp + SMS in one integration |
| Infrastructure | Docker Compose + GitHub Actions | Local dev parity, automated CI |

---

## System architecture

```
                        ┌─────────────────────┐
                        │   Worker PWA (React) │
                        │   Gagan Prakash · UI │
                        └──────────┬──────────┘
                                   │ HTTPS
                        ┌──────────▼──────────┐
                        │  FastAPI Backend     │
                        │  Arghajit Deb · BE  │
                        └──┬──────┬──────┬───┘
                           │      │      │
              ┌────────────▼─┐ ┌──▼───┐ ┌▼──────────────┐
              │  MySQL DB    │ │Redis │ │  ML Layer      │
              │  Workers     │ │Trigger│ │  XGBoost       │
              │  Policies    │ │State │ │  Isolation     │
              │  Claims      │ └──────┘ │  Forest        │
              └──────────────┘         │  Prophet        │
                                       └────────────────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
   ┌──────▼─────┐  ┌───────▼──────┐  ┌─────▼──────┐
   │ Open-Meteo │  │  Razorpay    │  │   Twilio   │
   │ CPCB / IMD │  │  (UPI mock)  │  │  WhatsApp  │
   └────────────┘  └──────────────┘  └────────────┘
```

---

## Repository structure

```
zerodown/
├── README.md
├── docker-compose.yml
│
├── frontend/
│   ├── public/manifest.json           # PWA manifest
│   └── src/
│       ├── pages/
│       │   ├── Onboarding.jsx         # 3-step registration
│       │   ├── WorkerDashboard.jsx    # Coverage status, payout history
│       │   ├── PolicyView.jsx         # Active plan, next week's premium
│       │   ├── ClaimsLog.jsx          # Claim status with plain-language notes
│       │   └── AdminDashboard.jsx     # Insurer: heatmap, loss ratios, forecast
│       └── components/
│           ├── ZoneMap.jsx            # Leaflet disruption zone overlay
│           └── PremiumCard.jsx        # Weekly premium breakdown widget
│
├── backend/
│   ├── main.py
│   ├── requirements.txt
│   └── app/
│       ├── routes/
│       │   ├── workers.py             # Registration, profile, OTP
│       │   ├── policies.py            # Policy creation and weekly renewal
│       │   ├── claims.py              # Claim status, payout initiation
│       │   └── triggers.py            # Real-time trigger monitoring loop
│       ├── ml/
│       │   ├── risk_profiler.py       # XGBoost zone risk model
│       │   ├── fraud_detector.py      # Isolation Forest anomaly scorer
│       │   ├── premium_engine.py      # Weekly premium calculator
│       │   └── disruption_forecast.py # Prophet next-week prediction
│       └── integrations/
│           ├── weather.py             # Open-Meteo + IMD wrapper
│           ├── aqi.py                 # CPCB feed parser
│           ├── payments.py            # Razorpay sandbox wrapper
│           └── notifications.py       # Twilio WhatsApp + SMS
│
└── ml/
    ├── notebooks/
    │   ├── 01_zone_risk_model.ipynb
    │   ├── 02_fraud_detection.ipynb
    │   └── 03_premium_forecasting.ipynb
    └── models/                        # Serialized .joblib artifacts
```

---

## Development timeline

### Phase 1 · March 4–20 — complete ✅
Persona selected with justification. Parametric trigger set defined with actual data sources. Weekly premium formula specified with worked example. ML approaches chosen. Tech stack finalized. Repository structured.

### Phase 2 · March 21–April 4
Worker registration and onboarding flow. FastAPI backend with MySQL schema. Open-Meteo and CPCB integrations connected. Risk profiling model on mock zone data. Dynamic weekly premium calculator. Isolation Forest fraud detector in the claim pipeline. Razorpay sandbox payout simulation. Full automated trigger → fraud check → payout flow tested end-to-end.

### Phase 3 · April 5–17
GPS spoofing detection added to fraud module. WhatsApp plain-language claim messages via LLM. Worker dashboard: earnings protected this week, payout history, next week's premium preview. Admin dashboard: zone risk heatmap, loss ratio tracker, 7-day disruption forecast. 5-minute walkthrough video demonstrating a live weather simulation → automatic claim → UPI payout. Final pitch deck.

---

## Phase 1 demo video
> Link to be added before March 20 end of day

---

## Team
| Name | Role |
|---|---|
| Rishi Kumar Mandal | Planning & Execution |
| Arghajit Deb | AI Integration & Backend |
| Gagan Prakash | UI |

---

*Built for Guidewire DEVTrails 2026 · Seed. Scale. Soar.*
