# TEA CRM Dashboard & Analysis
**Flow:** Experimental — Travel_Experiences_Location_based  
**Flow ID:** `6a295838b4cecb7392eeb7a5`  
**Live Dashboard:** https://adityageda-cm.github.io/CRM_Dashboard_Exp/  
**Period:** Jun 10 – Aug 13, 2026

---

## Repo Structure

```
├── index.html / dashboard.html     ← Live CRM dashboard (GitHub Pages)
├── data/
│   ├── dashboard_data.json         ← Embedded dashboard data (periods, AB, slots, daily)
│   └── tea_crm_daily_pulled.json   ← Raw daily MoEngage API pull (17 exact days)
└── analysis/
    ├── TEA_CRM_LeakageAnalysis_v2.xlsx       ← Main analysis (latest)
    ├── TEA_CRM_Attribution_Analysis.xlsx     ← Attribution deep-dive
    └── TEA_CRM_Daily_Flow_Data.xlsx          ← 65-day daily tracker
```

---

## Dashboard Sections

| Section | What it shows |
|---|---|
| S1 — KPI Summary | People entered, msgs sent, seen rate, clicks, conversion |
| S2 — Channel Lanes | P1 / P2 / WA / FP sent → seen → clicked per period |
| S3 — A/B Split | Engaged group vs held-out (Global Control Group) |
| S4 — Efficiency | CTR%, seen%, cost-per-click estimates |
| S5 — City Breakdown | Top cities by cohort trigger volume |
| S6 — Slot Broadcasts | Time-slot distribution of sends |

**Periods:** Today / Yesterday / This Week / All Time / Aug 11 (spike) / Aug 10

---

## Excel Analysis Files

### TEA_CRM_LeakageAnalysis_v2.xlsx ← START HERE
5 tabs covering the full picture:

| Tab | Contents |
|---|---|
| 1. Master Funnel (Daily) | 13 days MoEngage + DB funnel: trips → msgs → clicks → app opens → TEA → booking |
| 2. Channel Overlap & Leakage | Same user gets P1+P2+WA+FP (~5-6 msgs/person). Wasted msgs per day. |
| 3. CRM vs Organic TEA | DB confirmed Aug 8-12: CRM drives 0.28% of tea_city_view. Merch+Home = 70%+ |
| 4. MoEngage Channel Analytics | UNIQUE Push + WA: CTR, seen%, reachable users per day |
| 5. Summary & Actions | 12 findings (green/red) + 5 ranked fixes with owners |

### TEA_CRM_Attribution_Analysis.xlsx
| Tab | Contents |
|---|---|
| 1. CRM Daily Funnel | MoEngage delivery + DB post-click funnel Aug 1-12 |
| 2. Leakage Waterfall | 10,071 msgs → 1,657 people → 1 tea_city_view → 0 bookings |
| 3. TEA Source Attribution | Aug 10: HomePageEntry=135, merch=133, Push=1 of 356 total |
| 4. User Journey Sample | Second-by-second trace of the 1 user who reached tea_city_view |
| 5. Efficiency & Fixes | KPIs vs benchmarks, 5 prioritised fixes |

### TEA_CRM_Daily_Flow_Data.xlsx
65 rows (Jun 10 – Aug 13). Green = exact from MoEngage API. Grey = export needed from MoEngage UI (API only retains ~15 days of split_stage_stats history).

---

## Key Finding

**MoEngage conversion goal CVR = 0.0% on ALL 13 days. DB confirms: 1 user/day reaches tea_city_view via Push. 0 bookings.**

Post-click journey (Aug 10, DB confirmed):

```
10,071 msgs sent
  → 1,657 unique people
    → 1,371 app opens (Push attr)
      → 919 deep link fired  ← link IS received
        → 1,083 land on home screen  ← routing is broken
          → 241 click travel tab (manually)
            → 31 see TEA module
              → 1 reaches tea_city_view
                → 0 bookings
```

**Root cause:** `dynamic_link_executed` fires but the app drops users on home+mpin instead of the TEA city page. Engineering fix needed: persist deep link destination across mpin auth flow.

---

## Data Sources

| Source | Tool | Limitation |
|---|---|---|
| MoEngage split_stage_stats | MoEngage MCP API | ~15 days retention only |
| MoEngage flow summary (trips) | MoEngage MCP API | 90-day window, UNIQUE metric |
| MoEngage channel analytics | MoEngage MCP API | Push+WA separately |
| Databricks percept_event_all | Databricks MCP | Single-day queries only (multi-day timeout) |
| DB attribution key | `get_json_object(entry, '$.properties.source_attribute_type') = 'Push'` | |

---

## Stage IDs (split_stage_stats)

| Channel | Primary Stage ID | Branch names |
|---|---|---|
| P1 (Push #1) | `i7FAX4Ixd6` | Per + Simple |
| P2 (Push #2) | `wPxX7rS3U_` | Personalised + Simple |
| WhatsApp | `fcFNy-V4wW` | Personalised + Simple |
| Final Push | `cFnfJi0SVG` | Personalised + Simple |

**Formula:** `sent = engaged_group.total + global_control_group.total` (both branches)

---

*Last updated: Aug 14, 2026*
