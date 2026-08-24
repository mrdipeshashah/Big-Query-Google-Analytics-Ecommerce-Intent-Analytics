# Landing Page Performance & Intent Matrix Dashboard

An end-to-end Looker Studio documentation guide designed to track, analyze, and surface user landing page intent, channel attribution, and funnel conversion performance for GA4 e-commerce platforms.

### Methodology & Strategic Context

* **The UA vs. GA4 Metric Pivot:** This architecture uses Universal Analytics (UA) single-page exit principles to solve GA4's native ambiguity around time-on-page timers. It evaluates traffic strictly on a session level: **Did the landing session advance past the initial page, or did it exit without taking a step forward?**
* **PDP Intent & Multi-Page Funnels (80% Traffic Allocation):** For e-commerce stores where ~80% of acquisition traffic lands directly on Product Detail Pages (PDPs), traditional single-page metrics miss the full picture. This model tracks whether a session initiated on a landing page produced downstream progression—capturing users who land on `PDP A`, browse through to `PDP B`, and convert.
* **Paid Media Campaign Diagnostic:** By measuring immediate session drop-off at Step 1 of the funnel, this framework gives media buyers an instant diagnostic tool within 24–48 hours of launching a campaign. Seeing a **70% vs. 20% landing drop-off** immediately isolates whether campaign performance issues stem from poor ad-to-landing relevance versus downstream pricing or checkout friction.

---

## 📌 Dashboard Overview

The **Landing Page Performance & Intent Matrix** provides an executive-to-granular view of landing page behavior by categorizing page types, user intent, traffic channels, and device categories.

* **Top Row:** Executive KPI Scorecards (Sessions, Engaged Sessions, Revenue, Rev/Eng Session)
* **Middle Left:** Revenue Share by Category (Donut Chart)
* **Middle Right:** Engagement & Conversion Rate by Page Type (Grouped Bar Chart)
* **Center Row:** 30-Day Active Intent & Engagement Trend (Time-Series Combo Chart)
* **Bottom Row:** Landing Page Performance Matrix (Table with Hierarchical Drill-Down)

---

## 🧮 Looker Studio Calculated Fields Suite

### 1. Custom Channel Grouping (Calculated Dimension)
Add a **Calculated Field** named `Channel Grouping`:

```sql
CASE 
  WHEN REGEXP_MATCH(LOWER(medium), '^(cpc|ppc|paidsearch)$') THEN 'Paid Search'
  WHEN REGEXP_MATCH(LOWER(medium), '^(paidsocial|paid_social|meta_paid|social_paid)$') THEN 'Paid Social'
  WHEN LOWER(medium) = 'organic' THEN 'Organic Search'
  WHEN REGEXP_MATCH(LOWER(medium), '^(organicsocial|organic_social|social)$') THEN 'Organic Social'
  WHEN LOWER(medium) = 'referral' THEN 'Referral'
  WHEN REGEXP_MATCH(LOWER(medium), '^(email|newsletter)$') THEN 'Email'
  WHEN LOWER(medium) = 'affiliate' THEN 'Affiliates'
  WHEN (source IS NULL OR source = '(direct)' OR source = '(not set)') AND (medium IS NULL OR medium = '(none)' OR medium = '(not set)') THEN 'Direct'
  ELSE 'Unassigned'
END
```


### 2. Metric Calculations

| Calculated Field Name | Looker Studio Formula | Data Type |
| :--- | :--- | :--- |
| **Page Engagement Rate** | SUM(engaged_sessions) / SUM(total_sessions) | Percent |
| **Custom Bounce Rate** | SUM(bounced_sessions) / SUM(total_sessions) | Percent |
| **Revenue Per Session (RPS)** | SUM(total_revenue) / SUM(total_sessions) | Currency |
| **Revenue Per Engaged Session** | SUM(total_revenue) / SUM(engaged_sessions) | Currency |
| **Landing Page Conversion Rate** | SUM(total_purchases) / SUM(total_sessions) | Percent |
| **Transactions Per Engaged Session** | SUM(total_purchases) / SUM(engaged_sessions) | Percent |
| **Average Order Value (AOV)** | SUM(total_revenue) / SUM(total_purchases) | Currency |
| **% Revenue Share (For Charts)** | SUM(total_revenue) / SUM(total_revenue) | Percent |

---

## 📊 Looker Studio Chart Configuration

### 1. Executive KPI Scorecards
* **Metrics:** `Total Sessions`, `Page Engagement Rate`, `Total Revenue`, `Revenue Per Engaged Session`.

### 2. 30-Day Active Intent & Engagement Trend (Combo Chart)
* **X-Axis Dimension:** `event_date`
* **Series 1 (Bars):** `SUM(engaged_sessions)` mapped to **Left Y-Axis**.
* **Series 2 (Line):** `Page Engagement Rate` mapped to **Right Y-Axis** *(Set Right Axis Max to 1.0 / 100%)*.

### 3. Landing Page Performance Table (Drill-Down Setup)
* **Dimensions:**
  1. `Page Type` (Top Level)
  2. `Landing Page` (Sub-Level)
* **Drill-Down Switch:** Enabled.
* **Default Drill-Down Level:** Set to `Page Type`.
* **Chart Header:** Set to **Always Show** to display drill-down arrows.

### 4. Interactive Filters Controls
Add the following Drop-down controls at the top of the report:
* `Channel Grouping`
* `Source / Medium`
* `Device Category`
