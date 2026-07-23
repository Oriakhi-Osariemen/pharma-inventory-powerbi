# Pharmaceutical Inventory Management — Power BI Dashboard

An interactive, multi-page inventory analytics dashboard designed for pharmaceutical distributors. Built to surface critical operational intelligence across five key areas — expiry risk, stockout alerts, supplier performance, dead stock, and executive overview — in a clean, clinical design.

A live interactive HTML mockup is included so the dashboard can be explored directly in any browser without Power BI installed.

---

## Live demo

Open `pharma_dashboard_mockup.html` in any browser to interact with the full dashboard — all five pages, charts, tables, and navigation are fully functional.

---

## The business problem

Pharmaceutical distributors manage thousands of product batches across multiple warehouses. Without a unified analytical view, inventory teams cannot reliably answer:

- Which batches are expiring in the next 30–90 days — and what is the financial write-off exposure?
- Which products are about to stock out, and is a purchase order already in progress?
- Which suppliers are consistently late, and how much capital is tied up in their delays?
- Which products have not moved in over 60 days?
- How does stock compare across warehouses — and where can internal transfers replace costly new orders?

This dashboard answers all five questions in a single interface.

---

## Dashboard pages

### Page 1 — Executive Summary
The homepage. Designed to be the first screen an inventory manager opens each morning.

**Contains:**
- Critical alert strip flagging issues requiring immediate action
- Four headline KPIs: total inventory value, batches expiring within 90 days, products below reorder point, and overall supplier on-time rate
- Inventory value breakdown by product category (bar chart)
- Stock status distribution across all batches (donut chart)
- Warehouse snapshot cards showing stock value and health status for all five locations
- 30-day stock movement trend (inbound vs outbound line chart)

---

### Page 2 — Expiry Risk
Identifies at-risk batches before they become write-offs.

**Contains:**
- KPI cards split by urgency: expired, critical (≤30 days), high (31–60 days), medium (61–90 days)
- Financial exposure by urgency tier (horizontal bar chart)
- At-risk value by product category (pie chart)
- Full batch-level detail table with product, warehouse, quantity, expiry date, days remaining, stock value, and urgency status

**Key insight from the data:** Two batches of Insulin Glargine account for €33,300 of expired or near-expired stock — the single largest write-off risk in the portfolio.

---

### Page 3 — Stockout Alerts
Gives the procurement team a prioritised action list before stockouts happen.

**Contains:**
- KPI cards: active stockouts, products below reorder with no PO raised, products below reorder with a PO already in flight
- Average days of stock remaining across at-risk products
- Days of stock remaining per at-risk product (bar chart)
- Stock status split — adequate vs below reorder vs stockout (donut chart)
- Reorder alert table showing quantity on hand, reorder point, reorder quantity, days remaining, and PO status

**Key insight:** Paracetamol 500mg is in stockout in Berlin with no purchase order raised — the most urgent procurement action in the dataset.

---

### Page 4 — Supplier Scorecard
Ranks all suppliers by delivery performance to inform contract reviews.

**Contains:**
- KPI cards: excellent suppliers (≥95% on time), suppliers needing improvement, poor-performing suppliers, and total capital at risk from delayed orders
- On-time delivery rate progress bars for all five suppliers, colour-coded by performance tier
- Value at risk by supplier (bar chart)
- Full purchase order detail table with order date, expected delivery, actual delivery, delay in days, order value, and status

**Key insight:** BioVantage Ltd has delivered 0% of its orders on time, with €6,637 of undelivered stock across two delayed POs — both for cold-chain products (Insulin and Hepatitis B Vaccine) that are also approaching stockout.

---

### Page 5 — Dead Stock & Capital
Surfaces inventory tying up capital without generating throughput.

**Contains:**
- KPI cards: dead stock batches (no movement recorded), slow-moving batches (no dispatch in 60+ days), total capital tied up, and internal transfer opportunities identified
- Dead and slow-moving stock value by product category (bar chart)
- Days since last dispatch per at-risk batch (bar chart)
- Full detail table with last dispatch date, days idle, units dispatched in last 90 days, stock value, and classification

**Key insight:** A batch of 1,800 Insulin Glargine pens in Paris Central has no movement on record, representing €33,300 in idle capital — and the batch is already expired.

---

## Design decisions

| Choice | Rationale |
|---|---|
| Cream + burgundy palette | Clinical authority feel without the sterility of all-white medical dashboards |
| Sage green as the healthy signal | Avoids the generic teal; reads immediately as "safe" in a medical context |
| Left sidebar navigation | Mirrors the Power BI multi-page file structure exactly |
| Alert strip on the homepage | Ensures critical issues are never buried — visible before any filtering |
| Progress bars for supplier rates | More readable than a bar chart when comparing five named entities |
| Horizontal bar charts for financial exposure | Long product names read better on the Y-axis |

---

## Data source

The dashboard is built on the synthetic dataset from the companion SQL project (`pharma-inventory-sql`). Data covers:

| Table | Records |
|---|---|
| Products | 10 |
| Suppliers | 5 |
| Warehouses | 5 |
| Inventory batches | 15 |
| Stock movements | 15 |
| Purchase orders | 11 |

No real patient or commercial data is used. All figures are generated for portfolio demonstration purposes.

---

## How to replicate in Power BI

1. Run the SQL project (`pharma_inventory_analysis.sql`) against your Snowflake instance to populate the tables
2. Connect Power BI Desktop to Snowflake using the native connector (Get Data → Snowflake)
3. Import the following views as your data model: `raw_orders`, `raw_inventory`, `raw_products`, `raw_suppliers`, `raw_warehouses`, `raw_purchase_orders`
4. Recreate the five pages using this mockup as your design reference
5. Apply the colour theme: primary `#6B1E2E`, background `#F8F4EE`, positive signal `#7A9E7E`, warning `#C47A2B`, critical `#B53B3B`

### Recommended Power BI measures (DAX)

```dax
-- Days to expiry
Days To Expiry = DATEDIFF(TODAY(), inventory[expiry_date], DAY)

-- Stock value
Stock Value EUR = inventory[quantity_on_hand] * RELATED(products[unit_cost_eur])

-- On-time delivery rate
On Time Rate % =
DIVIDE(
    COUNTROWS(FILTER(purchase_orders, purchase_orders[actual_delivery] <= purchase_orders[expected_delivery])),
    COUNTROWS(FILTER(purchase_orders, purchase_orders[status] = "DELIVERED"))
)

-- Days of stock remaining
Days Of Stock Remaining =
DIVIDE(
    SUM(inventory[quantity_on_hand]),
    [Avg Daily Consumption]
)

-- Capital at risk (delayed POs)
Capital At Risk EUR =
SUMX(
    FILTER(purchase_orders, purchase_orders[status] IN {"DELAYED", "PENDING"}),
    purchase_orders[quantity_ordered] * purchase_orders[unit_cost_eur]
)
```

---

## Repository structure

```
pharma-inventory-powerbi/
├── pharma_dashboard_mockup.html    # Interactive browser demo (open this first)
├── README.md                       # This file
└── screenshots/
    ├── page_1_overview.png
    ├── page_2_expiry_risk.png
    ├── page_3_stockout_alerts.png
    ├── page_4_supplier_scorecard.png
    └── page_5_dead_stock.png
```

---

## Related project

This dashboard is the visualisation layer for the companion SQL project:
**[pharma-inventory-sql](https://github.com/YOUR_USERNAME/pharma-inventory-sql)** — schema design, seed data, and five analytical queries solving the same business problems in pure SQL.

---

## GitHub setup

**Repository name:** `pharma-inventory-powerbi`

**Description:**
```
Interactive Power BI dashboard for pharmaceutical inventory management — expiry risk, stockout alerts, supplier performance, and dead stock analysis.
```

**Topics:**
```
power-bi  dashboard  data-visualisation  inventory-management  healthcare  sql  portfolio  analytics
```

---

## Author

Osariemen Oriakhi — Data & Finance Analyst, Paris
[LinkedIn](https://linkedin.com/in/your-profile) · [GitHub](https://github.com/your-username)
