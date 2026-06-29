
# MereExpress

**MereExpress** = Just Express Food.  
**Tagline:** Multi-tenant food delivery marketplace. See menus from restaurants near you, order, and get delivery to your location at a cost.

`Next.js + FastAPI + Postgres + PostGIS + Docker + Security + DevOps + Data + "Smart" features`  
**No AI in the name. No AI in the pitch.**

---

### **0. EXECUTIVE SUMMARY** 
`MereExpress` = `Glovo` + `Jumia Food` but built for Kampala, 5% fee vs 25%. 
**Goal V1:** Get 3 restaurants live in Nakawa/Wandegeya. Show: Browse → Order → Track → Deliver → Report. 
**Success Metric:** 1 order placed end-to-end in <5 mins with live rider map.

---

### **1. VISION, USERS + SCOPE** 

**Vision:** Kill WhatsApp food orders. Make restaurant discovery + delivery feel like "just press and it comes".

**Primary Users + JTBD**

| **User** | **Job To Be Done** | **Access Device** |
| --- | --- | --- |
| **Customer** | Find food near me, order, pay delivery fee, track rider to gate | Mobile PWA |
| **Restaurant Owner** | Manage menu, see orders, analytics, manage riders | Desktop PWA |
| **Cook** | See new orders, mark Cooking → Ready | Mobile/Tablet PWA |
| **Rider** | Accept order, navigate, update location live, mark Delivered | Mobile PWA |
| **Admin** | Onboard restaurants, 5% fee, handle disputes, see platform data | Web Dashboard |

**Scope V1:** Kampala only. Radius search 3km. Cash + Momo + Wallet. English only. 
**Out of Scope V1:** Subscriptions, Loyalty points, USSD, iOS App Store.

---

### **2. PROBLEMS → MODULE MAP** 

| **#** | **Pain in Market** | **MereExpress Module** | **Measurable Output** |
| --- | --- | --- | --- |
| **1** | "Who has Rolex near me?" | `Live Menu Board` | PostGIS `radius=3km` list with filters |
| **2** | 4 friends, 1 pays, fights after | `Split Bill` | 1 Order → N Momo requests, status dashboard |
| **3** | "I’m at the gate" x10 calls | `GPS Pin + Live Map` | Customer pin + note. Rider ETA via WebSocket |
| **4** | Restaurant doesn’t know what sells | `Restaurant Analytics` | CSV: Top items, Peak hours, Stock <10 alerts |
| **5** | Cash lost, no receipts | `Wallet + Momo/Airtel` | Digital receipt + balance. First order 10% off |
| **6** | Food wasted at 9pm | `Smart Deals` | 9pm RQ cron → 15% off. Push to 3km radius |

---

### **3. SYSTEM ARCHITECTURE + DATA FLOW** 
[ Customer PWA Next.js ]  
[ Owner Dashboard ]      <--> [ FastAPI Gateway + WebSockets ]  <--> [ Postgres 16 + PostGIS + pgvector ]
[ Rider PWA Map ]             |                                        |
                              +--> [ Redis + RQ Jobs ]                 +--> [ MinIO/S3 for photos ]
                              +--> [ Momo/Airtel API ] 
                              +--> [ Africa’s Talking SMS ]
**Architecture Principles:**
1.  **Multi-tenant:** Every `restaurant` = 1 `org`. All tables have `org_id`. Middleware enforces `WHERE org_id = current_org`.
2.  **PWA-first:** Installable. Offline menu cache. Orders queue locally, sync when online.
3.  **Realtime:** WebSocket channel `/ws/orders/{order_id}` for status + rider lat/lng.
4.  **Location:** PostGIS `GEOGRAPHY` type. Query: `ST_DWithin(location, point, 3000)` for 3km.
5.  **Audit Everything:** `audit_logs` for order status change, payment confirm, login.

---

### **4. TECH STACK V1** 
Frontend:  Next.js 14 + TS + Tailwind + PWA + Leaflet Maps + WebSockets
Backend:   Python 3.11 + FastAPI + Pydantic v2 + SQLAlchemy 2.0 
DB:        Postgres 16 + PostGIS + pgvector 
Realtime:  Redis + WebSockets + RQ
Files:     MinIO S3 for food photos
Payments:  Momo/Airtel + Wallet + Stripe for restaurant $19/mo subs
SMS:       Africa’s Talking for "Rider arriving" alerts
DevOps:    Docker, docker-compose, GitHub Actions CI
Auth:      JWT + bcrypt + RBAC

---

### **5. QUICK START - LOCAL DEV** 
```bash
# 1. Clone and start everything
docker-compose up --build

# Services:
# api:     http://localhost:8000  -> FastAPI docs at /docs
# web:     http://localhost:3000  -> Customer + Owner + Rider PWA
# db:      Postgres + PostGIS + pgvector
# redis:   Redis + RQ
# minio:   http://localhost:9001  -> S3 for images
Reqs: Docker + Docker Compose

---

"SMART" FEATURES 
Smart Deals:  → Auto 15% off leftovers. Push to 3km radius. 
Smart Search:  → "cheap snacks" → vector menu matches. 
Smart Splits: Rules engine → Equal share + Momo link per friend. 
Auto Reports:  → Daily sales CSV, Peak hours, Top 5 items. 
ETA Engine: PostGIS distance / 15km/h = "Rider 7 mins away".

---

14-DAY BUILD PLAN
| **Week** | **Deliverable** | **Demo** |
| **W1 D1-D2** | `docker-compose + DB + Auth + RBAC` | Signup Owner, Customer, Rider |
| **W1 D3-D4** | `Restaurant CRUD + Menu CRUD + PostGIS search` | See 3km restaurants |
| **W1 D5** | `Order + OrderItems + Wallet mock` | Place order, see total |
| **W2 D6-D7** | `Rider PWA + WebSockets tracking` | Live map moves |
| **W2 D8** | `Split Bill + Momo mock` | 3 friends get pay links |
| **W2 D9** | `Smart Deals RQ + Reports CSV` | Download sales.csv |
| **W2 D10-D14** | `PWA offline, Polish, Deploy, Demo video` | End-to-end flow recorded |
---

V1 DEMO SCRIPT / ACCEPTANCE CRITERIA 
Owner signs up "Rolex Express" → Adds 3 menu items with photos.
Customer in Nakawa opens PWA → Sees "Rolex Express 1.2km away".
Customer orders 2x Rolex + 1 Soda = 18,000 UGX + 3,000 delivery.
Customer pins location + note: "blue gate".
Cook marks . Rider accepts → Location pings every 3s.
Customer sees "Rider 7 mins away" live.
Rider marks  → Customer rates 5 stars.
Owner downloads  → Shows "Rolex: 12 units".

---

LICENSE 
MIT License - built for portfolio + Kampala market.

---
Built by: @emmanuelmmiiro  
Status: V1.1 In Development 🚀

---

**NEXT MOVE BRO:** 
Want me to drop `docker-compose.yml` + `backend/app/models.py` + `backend/app/main.py` now so you can run `docker-compose up` in 2 mins?

Say `ship MereExpress` and we start Day 1 code.
