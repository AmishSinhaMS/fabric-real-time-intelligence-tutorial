# 🚲 Real-Time Intelligence Tutorial — Microsoft Fabric
### Bicycle Rentals | Streaming Data | Bronze → Silver → Gold

---

## 📋 Overview

Complete implementation of the official [Microsoft Fabric Real-Time Intelligence tutorial](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/tutorial-introduction) using simulated bicycle rental streaming data.

This solution demonstrates the full Real-Time Intelligence stack: **Eventhouse**, **Eventstreams**, **KQL queries**, **Update Policies**, **Materialized Views**, **Real-Time Dashboards**, and **Activator Alerts**.

### Architecture

```
┌─────────────────────────────────┐
│  Real-Time Hub                   │
│  "Bicycle Rentals" sample source │
└──────────┬──────────────────────┘
           │
           ▼
┌─────────────────────────────────┐
│  TutorialEventstream             │
│  + Manage Fields transform       │
│    (adds SYSTEM.Timestamp)       │
└──────────┬──────────────────────┘
           │  Eventhouse destination
           ▼
┌─────────────────────────────────────────────────┐
│  Tutorial Eventhouse / KQL Database              │
│                                                  │
│  🥉 Bronze: RawData                              │
│    Raw bike point events (string BikepointID)    │
│         │                                        │
│         ▼  Update Policy + TransformRawData()    │
│  🥈 Silver: TransformedData                      │
│    • BikepointID parsed to int                   │
│    • BikesToBeFilled = Empty_Docks - Bikes       │
│    • Action: bikes needed or "NA"                │
│         │                                        │
│         ▼  Materialized View                     │
│  🥇 Gold: AggregatedData                         │
│    Latest No_Bikes per BikepointID               │
│    (always up-to-date, pre-aggregated)           │
└──────────┬──────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────────────┐
│  TutorialDashboard (Real-Time Dashboard)         │
│  • Column chart: Bikes by station                │
│  • Map: Bike locations with lat/lon              │
│  • Activator alert: High bike count (>30)        │
└─────────────────────────────────────────────────┘
```

---

## 🚀 Setup Guide

### What's Pre-Built (via API)
| Item | Status |
|------|--------|
| **Workspace:** Real-Time Intelligence Tutorial | ✅ Created |
| **Eventhouse:** Tutorial (+ KQL Database) | ✅ Created |
| **KQL Bronze:** RawData table (120 rows sample data) | ✅ Created + populated |
| **KQL Silver:** TransformedData table (update policy active) | ✅ Created + auto-populated |
| **KQL Gold:** AggregatedData materialized view | ✅ Created + auto-populated |
| **KQL Function:** TransformRawData() | ✅ Created |
| **Eventstream:** TutorialEventstream | ✅ Configured (CustomEndpoint → Eventhouse) |
| **Dashboard:** TutorialDashboard | ✅ Configured (4 KQL queries, 30s auto-refresh) |

### Steps to Complete in Fabric Portal

#### Step 1: Configure Eventstream Source
1. Open **TutorialEventstream** in the workspace
2. Click **Edit**
3. On the **Transform events or add destination** tile → select **Manage fields**
4. Click the pencil icon:
   - **Operation name:** `TutorialTransform`
   - Click **Add all fields**
   - Click **+ Add Field** → **Built-in Date Time Function** → `SYSTEM.Timestamp()`
   - **Name:** `Timestamp`
   - Click **Add** → **Save**

> ⚠️ First, you need to add the source: Go to **Real-Time Hub** → **Add data** → **Sample scenarios** → **Bicycle rentals** → connect it to TutorialEventstream

#### Step 2: Add Eventhouse Destination
1. Point to the right edge of **TutorialTransform** → click green **+** icon
2. Select **Destinations** → **Eventhouse**
3. Configure:
   - **Data ingestion mode:** Event processing before ingestion
   - **Destination name:** `TutorialDestination`
   - **Eventhouse/KQL Database:** `Tutorial`
   - **Destination table:** Select existing → `RawData`
   - **Input data format:** Json
4. Check **Activate ingestion**
5. Click **Save** → **Publish**

#### Step 3: Create Real-Time Dashboard
1. Open **Tutorial_queryset** (in KQL Database)
2. Run the column chart query:
   ```kql
   AggregatedData
   | sort by BikepointID
   | render columnchart with (ycolumns=No_Bikes, xcolumn=BikepointID)
   ```
3. Click **Save to dashboard** → **New Real-Time Dashboard** → Name: `TutorialDashboard`

#### Step 4: Add Map Tile
1. In the dashboard, click **New tile**
2. Run: `RawData | where Timestamp > ago(1h)`
3. Click **+ Add visual** → Visual type: **Map**
4. Configure: Latitude/Longitude columns, Label: BikepointID

#### Step 5: Set Dashboard Alert
1. On the column chart tile → **Set Alert**
2. Configure:
   - **Rule name:** `High Bike Count Alert`
   - **Run query every:** 5 minutes
   - **When:** BikepointID **Is greater than** 30
   - **Action:** Message to individuals → your Teams account

#### Step 6: Set Eventstream Alert
1. Go to **Real-Time Hub** → open **TutorialEventstream**
2. Click **Set alert**
3. Configure:
   - **Rule name:** `TutorialRule`
   - **When:** No_Bikes **Is equal to** 0
   - **Action:** Message to your Teams account
   - **Save as:** `TutorialActivator`

---

## 📁 Repository Structure

```
fabric-real-time-intelligence-tutorial/
├── README.md                          # This guide
├── kql/
│   └── setup_script.kql               # Complete KQL setup (tables, function, policy, view)
├── queries/
│   └── sample_queries.kql             # 8 sample KQL queries for exploration
├── .gitignore
└── LICENSE
```

---

## 🔍 KQL Medallion Structure

### Bronze: `RawData`
| Column | Type | Description |
|--------|------|-------------|
| BikepointID | string | Bike station ID (e.g., "BikePoints_123") |
| Street | string | Street name |
| Neighbourhood | string | Area name |
| Latitude | dynamic | GPS latitude |
| Longitude | dynamic | GPS longitude |
| No_Bikes | long | Current bike count |
| No_Empty_Docks | long | Available empty docks |
| Timestamp | datetime | Event timestamp |

### Silver: `TransformedData` (via Update Policy)
| Added Column | Type | Logic |
|-------------|------|-------|
| BikepointID | int | Parsed from "BikePoints_123" → 123 |
| BikesToBeFilled | long | No_Empty_Docks - No_Bikes |
| Action | string | If BikesToBeFilled > 0 → count, else "NA" |

### Gold: `AggregatedData` (Materialized View)
| Column | Type | Description |
|--------|------|-------------|
| BikepointID | int | Station ID |
| Timestamp | datetime | Latest event time |
| No_Bikes | long | Most recent bike count |

---

## 🔗 References

- [Official Tutorial](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/tutorial-introduction)
- [Tutorial KQL Script (GitHub)](https://github.com/microsoft/fabric-samples/blob/main/docs-samples/real-time-intelligence/tutorial-commands-script.kql)
- [What is Real-Time Intelligence?](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/overview)
- [Eventhouse Overview](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/eventhouse)
- [KQL Reference](https://learn.microsoft.com/en-us/kusto/query/)

---

> 📝 **Workspace:** Real-Time Intelligence Tutorial | **Data:** Bicycle Rentals (streaming) | **Last updated:** April 2026

