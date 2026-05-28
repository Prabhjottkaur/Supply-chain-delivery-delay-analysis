# Supply-chain-delivery-delay-analysis
### What factors are driving customer delivery delays?
---

## Objective

Late deliveries cost businesses customers, revenue, and trust but fixing them requires knowing exactly where and why delays happen. This project analyzes 169,000 orders from a global e-commerce supply chain to identify which factors responsible for delivery delays, and where the business should focus to reduce them.

The analysis goes beyond simply reporting which regions are delayed. It systematically tests multiple factors — shipping mode, region, product category, customer segment to determine what actually drives delay patterns versus what is just noise.

---

## Business Question

> **What factors are driving customer delivery delays, and where should the business focus to reduce them?**

---

## Sub Questions

| # | Sub-Question | Purpose |
|---|---|---|
| Q1 | Which shipping mode causes the most delays? 
| Q2 | Which regions have the highest delay rates? 
| Q3 | Which product categories are most delay-prone? 
| Q4 | How do shipping mode and region interact? 
| Q5 | Are delays increasing over time? 
| Q6 | Do order volume spikes lead to more delays? 
| Q7 | Which customer segments are most affected? 
---

## Dataset

| Field | Detail |
|---|---|
| Source | DataCo Supply Chain Dataset — Kaggle |
| Raw Records | 180,000 orders |
| After Deduplication | 169,000 orders |
| Regions | 23 global regions |
| Key Columns | delivery_status, days_shipping_real, days_shipment_scheduled, order_region, shipping_mode, category_name, customer_segment, order_date |

Data was deduplicated and cleaned in Excel Power Query before SQL analysis.

---

### Core Metrics

Two metrics were derived to measure delays:

```sql
-- Frequency: was it late?
SUM(CASE WHEN delivery_status = 'Late Delivery' THEN 1 ELSE 0 END) * 100.0 / COUNT(*) AS late_delivery_rate

-- Severity: how late was it?
(days_shipping_real - days_shipment_scheduled) AS delivery_delay_days
```

Using both metrics together gives a complete picture frequency tells you how often delays happen, severity tells you how bad they are.
------
### SQL View for Reuse

```sql
CREATE VIEW delivery_analysis AS
SELECT
    order_date, order_region, category_name,
    shipping_mode, customer_segment,
    delivery_status, late_delivery_risk,
    (days_shipping_real - days_shipment_scheduled) AS delivery_delay_days
FROM order_fulfillment;
```

All segmentation queries reuse this view without repeating the calculation logic.

---

## Key Findings

### Finding 1 — Shipping Mode (Most Important)

| Shipping Mode | Late Delivery Rate | Avg Delay Days | Total Orders |
|---|---|---|---|
| First Class | **95.3%** | 1.00 days late | 26,051 |
| Second Class | **76.8%** | 1.99 days late | 32,939 |
| Same Day | 45.7% | 0.48 days late | 9,118 |
| Standard Class | **38.1%** | On time / early | 101,041 |

**The finding:**
First Class shipping, the premium, expensive option has a 95.3% late delivery rate. Standard Class, the cheapest, slowest option is the most reliable. This holds consistently across every region in the dataset.

The likely cause: premium shipping modes have much tighter scheduled delivery windows, making it easier to miss the deadline even with small operational delays. Standard Class has looser windows that naturally absorb delays.

---

### Finding 2 — Regional Analysis

All 23 regions cluster between **49% and 58% late delivery rate** — an extremely narrow band.

| Metric | Value |
|---|---|
| Highest region | Central Africa — 57.6% |
| Lowest region | Canada — 49.4% |
| Range across all regions | 49% to 58% |

**The insight: If region were driving delays, some regions would be dramatically worse than others. The uniformity confirms the delay problem is structural and systemic not geographic. Fixing a specific region will not solve the underlying issue.

---

### Finding 3 — Shipping Mode × Region Interaction

The shipping mode pattern holds universally across all 23 regions:
- First Class is always 92–100% late regardless of region
- Standard Class is always 29–42% late regardless of region

Selected extremes:
- **Central Asia First Class:** 100% late rate — every single order arrived late
- **East Africa First Class:** 98.8% late
- **Canada Standard Class:** 29.1% late — best performing combination in the dataset

**This cross-validation confirms no region is making First Class reliable it is a scheduling and operations issue, not a geography issue.

---

### Finding 4 — Overall Scale

| Metric | Value |
|---|---|
| Total Orders Analyzed | 169,000 |
| Overall Late Delivery Rate | 54.9% |
| Avg Delay Days | 0.57 days |

More than half of all orders arrive late. This is not a minor operational issue it affects the majority of customers.

---

## Conclusion

> Delivery delays are not a regional problem they are a structural shipping mode problem. First Class and Second Class shipping systematically fail to meet their own scheduled delivery windows across all 23 regions. Standard Class consistently meets or beats its schedule. The fix is not geographic it is operational.

---

## Dashboard

Power BI dashboard 2 pages:

- **Page 1 Overview:** KPI cards (total orders, late delivery rate, avg delay days), time trend line chart, shipping mode donut chart.
- **Page 2 Deep Dive:** Shipping mode × region heatmap, category delay bar chart, customer segment bar chart.

---

## Tools Used

| Tool | Purpose |
|---|---|
| Excel Power Query | Deduplication, data cleaning |
| MySQL | View creation, segmentation queries across 7 sub-questions |
| Power BI | Interactive 2-page operational dashboard |

---
