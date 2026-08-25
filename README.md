# Landing Page Performance & Intent Matrix Dashboard

An end-to-end Looker Studio documentation guide designed to track, analyze, and surface user landing page intent, channel attribution, and funnel conversion performance for GA4 e-commerce platforms.

### Methodology & Strategic Context

* **The UA vs. GA4 Metric Pivot:** This architecture uses Universal Analytics (UA) single-page exit principles to solve GA4's native ambiguity around time-on-page timers. It evaluates traffic strictly on a session level: **Did the landing session advance into the e-commerce funnel, or did it drop off without taking action?**
* **PDP-to-Cart Funnel Logic (80% Traffic Allocation):** For e-commerce stores where ~80% of acquisition traffic lands directly on Product Detail Pages (PDPs), evaluating landing performance requires tracking transactional intent. This model measures whether a landing session successfully converted into an **Add to Cart** or **Purchase** step.
* **Paid Media Campaign Diagnostic:** By measuring immediate session drop-off before the Add to Cart step, this framework gives media buyers an instant diagnostic tool within 24–48 hours of launching a campaign. Seeing a **70% vs. 20% landing-to-cart drop-off** immediately isolates whether campaign performance issues stem from ad-to-landing relevance versus downstream pricing or checkout friction.

## Data Model & Metric Definitions (Full Dashboard Architecture)

### 1. Intent-Based Bounce Rate Model
* **Product Detail Pages (PDPs):**
  * **Non-Bounced Session:** A session starting on a PDP path where `add_to_cart = 1` (Transactional Intent).
  * **Bounced Session:** A session starting on a PDP path where `add_to_cart = 0`.
* **Product Listing Pages (PLPs / Collections):**
  * **Non-Bounced Session:** A session starting on a PLP path where `select_item = 1` or downstream `add_to_cart = 1` (Navigation/Discovery Intent).
  * **Bounced Session:** A session starting on a PLP path where neither `select_item` nor `add_to_cart` occurs.

> **Note on Model Trade-Offs:** 
> This is a session-level entry model designed for marketing acquisition performance. If a user lands on Product A, cross-shops, and purchases Product B, the session conversion is attributed to Product A's landing efficiency.

### 2. Future Rule Adjustments (Roadmap)
* **Compound Micro-Conversions:** If video tracking is added to PDPs, high-intent engagement can be gated using sequential conditions (e.g., `Video Watch >= 75%` **AND** `Add to Cart`).
* **Item-Scoped Funnel Model:** A secondary model can be built using unnested hit-level arrays (`items.item_name`) to track true SKU-level cross-selling and merchandising movement.

## Data Studio Calculated Fields Suite

### 1. Custom Channel Grouping (Calculated Dimension for Page 1)
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

| Page Scope | Calculated Field Name | Looker Studio Formula | Data Type |
| :--- | :--- | :--- | :--- |
| **Page 1** | Page Engagement Rate | `SUM(engaged_sessions) / SUM(total_sessions)` | Percent |
| **Page 1** | Custom Bounce Rate | `SUM(bounced_sessions) / SUM(total_sessions)` | Percent |
| **Page 1** | Revenue Per Session (RPS) | `SUM(total_revenue) / SUM(total_sessions)` | Currency |
| **Page 1** | Revenue Per Engaged Session | `SUM(total_revenue) / SUM(engaged_sessions)` | Currency |
| **Page 1** | Landing Page Conversion Rate | `SUM(total_purchases) / SUM(total_sessions)` | Percent |
| **Page 1** | Transactions Per Engaged Session | `SUM(total_purchases) / SUM(engaged_sessions)` | Percent |
| **Page 1** | Average Order Value (AOV) | `SUM(total_revenue) / SUM(total_purchases)` | Currency |
| **Page 1** | % Revenue Share (For Charts) | `SUM(total_revenue) / SUM(total_revenue)` | Percent |
| **Page 2** | PDP Custom Bounce Rate | `SUM(bounced_sessions) / SUM(total_sessions)` | Percent |
| **Page 2** | PDP Cart Addition Rate | `SUM(cart_sessions) / SUM(total_sessions)` | Percent |
| **Page 2** | PDP Session Conversion Rate | `SUM(purchase_sessions) / SUM(total_sessions)` | Percent |
| **Page 2** | Cart-to-Purchase Abandonment Rate | `(SUM(cart_sessions) - SUM(purchase_sessions)) / SUM(cart_sessions)` | Percent |
| **Page 2** | Cart-to-Purchase Conversion Rate | `SUM(purchase_sessions) / SUM(cart_sessions)` | Percent |

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
