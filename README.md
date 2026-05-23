 Project Summary
This project replicates a real-world data analyst workflow inside an automotive business.
Starting from four raw datasets (orders, vehicles, customers, dealerships), it builds a complete
analytics pipeline that answers 28 business questions across sales, customers, dealerships,
finance, and after-sales service — then visualizes the key KPIs in an executive Power BI dashboard.
Business context: Mahindra & Mahindra is one of India's largest vehicle manufacturers. This
project analyzes sales performance, customer behavior, dealership operations, financing patterns,
and service analytics — the exact work performed by DA/BI teams in the automotive industry.

🗂️ Project Structure
bashmahindra-automotive-business-analytics/
│
├── 📁 data/
│   ├── customers.csv          # Demographics, city, income group, customer type
│   ├── vehicles.csv           # Models, categories, fuel types, segments
│   ├── orders.csv             # Transactions: price, status, dates, payment type
│   └── dealerships.csv        # Dealer info, locations, sales targets
│
├── 📁 sql/
│   ├── SALES_ANALYTICS.sql          # 5 queries — revenue, models, trends, geography
│   ├── CUSTOMER_ANALYTICS.sql       # 5 queries — CLV, repeat buyers, demographics
│   ├── DEALERSHIP_ANALYTICS.sql     # 4 queries — performance, delays, targets
│   ├── FINANCE_ANALYTICS.sql        # 4 queries — loans, EMI, defaults, dependency
│   ├── SERVICE_ANALYTICS.sql        # 4 queries — servicing, satisfaction, retention
│   └── ADVANCED_SQL_ANALYTICS.sql   # 6 queries — cohorts, Pareto, EV growth, running totals
│
├── 📁 powerbi/
│   └── Mahindra_Executive_Dashboard.pbix
│
├── 📁 screenshots/
│   └── executive_dashboard.png
│
└── README.md

🎯 28 Business Questions Answered
#Business QuestionModuleSQL Concept1What is total business revenue?SalesAggregation + Filter2How many vehicles were sold?SalesCOUNT + Filter3Which vehicle model generates highest revenue?SalesCTE + DENSE_RANK4Which vehicle category sells the most?SalesCTE + Window Function5What is monthly revenue trend with MoM growth %?SalesCTE + LAG6Which states generate the highest revenue?SalesMulti-JOIN + DENSE_RANK7What is average customer age by vehicle category?Customer3-table JOIN + AVG8Which income groups buy premium vehicles?CustomerJOIN + GROUP BY + LIMIT9How many customers are repeat buyers?CustomerSubquery + HAVING10Who are the top 10 highest-value customers? (CLV)CustomerCTE + DENSE_RANK11How does Urban vs Rural revenue compare?CustomerJOIN + CASE filter12Which dealer generates highest revenue?DealershipCTE + ROW_NUMBER13Which dealers have delivery delays above 30 days?DealershipCTE chain + Date arithmetic14Which cities have the highest cancellation rates?DealershipCASE + DENSE_RANK15Which dealers achieved their sales targets?DealershipJOIN + CASE WHEN16What percentage of customers use loans?FinanceCASE + Ratio17What is the loan default rate?FinanceConditional aggregation18What is average EMI by vehicle category?Finance3-JOIN + AVG + ROUND19Which vehicle categories are most loan-dependent?FinanceLoan vs revenue ratio20Which vehicle models require the most servicing?ServiceJOIN + COUNT + ORDER BY21What is the average service cost by model?ServiceJOIN + AVG22Which dealerships have the lowest satisfaction scores?ServiceCTE + DENSE_RANK23What is the customer retention rate through servicing?ServiceCTE + HAVING + Subquery24What is the 2025 month-over-month growth rate?AdvancedLAG + EXTRACT + % calc25Which are the top 3 revenue models per state?AdvancedPARTITION BY + DENSE_RANK26Which monthly cohorts retained customers longest?AdvancedAGE() + DATE_TRUNC + cohort JOIN27What is the running cumulative revenue by month?AdvancedSUM() OVER (ORDER BY)28Which 20% of models drive 80% of revenue? (Pareto)AdvancedCumulative SUM + % threshold29What is Mahindra EV adoption growth year-over-year?AdvancedLAG + NULLIF + YoY %

🧠 SQL Showcase — 6 Key Queries
1. Month-over-Month Revenue Growth — LAG Window Function
sqlWITH rev AS (
    SELECT
        DATE_TRUNC('month', order_date)        AS month,
        ROUND(SUM(final_price), 0)             AS total_revenue_month
    FROM orders
    GROUP BY 1
)
SELECT
    TO_CHAR(month, 'MM-YYYY')                  AS monthly,
    total_revenue_month,
    LAG(total_revenue_month) OVER (
        ORDER BY TO_CHAR(month, 'MM-YYYY')
    )                                          AS previous_month_revenue,
    ROUND(
        (total_revenue_month - LAG(total_revenue_month) OVER (
            ORDER BY TO_CHAR(month, 'MM-YYYY'))
        ) / LAG(total_revenue_month) OVER (
            ORDER BY TO_CHAR(month, 'MM-YYYY')
        ) * 100.0, 1
    )                                          AS monthly_growth_pct
FROM rev;

💡 Business value: Pinpoints months where revenue declined — triggers investigation into supply, demand, or pricing factors.


2. Top 3 Models Per State — PARTITION BY
sqlWITH revenue AS (
    SELECT
        c.state,
        v.model_name,
        ROUND(SUM(o.final_price)::NUMERIC, 1)  AS total_revenue
    FROM orders o
    JOIN customers c USING (customer_id)
    JOIN vehicles v  USING (vehicle_id)
    WHERE o.order_status = 'Delivered'
    GROUP BY 1, 2
)
SELECT * FROM (
    SELECT *,
        DENSE_RANK() OVER (
            PARTITION BY state
            ORDER BY total_revenue DESC
        ) AS ranking
    FROM revenue
) t
WHERE ranking <= 3;

💡 Business value: Enables state-level marketing — each region gets product campaigns based on what actually sells there.


3. Customer Lifetime Value — Top 10
sqlWITH clv AS (
    SELECT
        customer_id,
        ROUND(SUM(final_price), 0)             AS total_revenue
    FROM orders
    WHERE order_status = 'Delivered'
    GROUP BY 1
)
SELECT customer_id, customer_name, city, total_revenue
FROM (
    SELECT
        c.customer_id, c.customer_name, c.city,
        r.total_revenue,
        DENSE_RANK() OVER (
            ORDER BY total_revenue DESC
        )                                      AS ranking
    FROM clv r
    JOIN customers c USING (customer_id)
) t
WHERE ranking <= 10;

💡 Business value: Identifies VIP customers for loyalty programs, priority service, and retention campaigns.


4. Cohort Retention Analysis
sqlWITH cohort AS (
    SELECT
        customer_id,
        DATE_TRUNC('month', MIN(order_date))   AS cohort_month
    FROM orders
    WHERE order_status = 'Delivered'
      AND EXTRACT('YEAR' FROM order_date) = 2024
    GROUP BY 1
),
dated AS (
    SELECT
        c.customer_id, c.cohort_month,
        (EXTRACT(YEAR  FROM AGE(DATE_TRUNC('month', o.order_date), c.cohort_month)) * 12
       + EXTRACT(MONTH FROM AGE(DATE_TRUNC('month', o.order_date), c.cohort_month))
        )                                      AS months_since_cohort
    FROM orders o
    JOIN cohort c USING (customer_id)
)
SELECT
    TO_CHAR(cohort_month, 'MM-YYYY')           AS cohort_month,
    months_since_cohort,
    COUNT(DISTINCT customer_id)                AS retained_customers
FROM dated
WHERE months_since_cohort > 0
GROUP BY 1, 2
ORDER BY 1, 2;

💡 Business value: Shows which acquisition campaigns produced the most loyal long-term customers.


5. Pareto Analysis — 80/20 Revenue Rule
sqlWITH total AS (
    SELECT
        vehicle_id,
        SUM(final_price)                       AS revenue,
        ROUND(SUM(final_price) * 100 /
            (SELECT SUM(final_price) FROM orders
             WHERE order_status = 'Delivered'), 2
        )                                      AS contribution_pct
    FROM orders
    WHERE order_status = 'Delivered'
    GROUP BY 1
),
cumulative AS (
    SELECT vehicle_id, revenue, contribution_pct,
        SUM(revenue) OVER (ORDER BY revenue DESC)           AS running_total,
        ROUND(SUM(revenue) OVER (ORDER BY revenue DESC)
            * 100 / SUM(revenue) OVER (), 2)                AS cumulative_pct
    FROM total
)
SELECT vehicle_id, contribution_pct, revenue, running_total, cumulative_pct
FROM cumulative
WHERE cumulative_pct <= 80
ORDER BY revenue DESC;

💡 Business value: Identifies which vehicle models to protect — the minority driving the majority of revenue.


6. EV Adoption YoY Growth — LAG + NULLIF
sqlWITH ev_sales AS (
    SELECT
        EXTRACT('YEAR' FROM o.order_date)      AS year,
        COUNT(o.order_id)                      AS total_ev_sales
    FROM orders o
    JOIN vehicles v USING (vehicle_id)
    WHERE o.order_status = 'Delivered'
      AND v.vehicle_category = 'EV'
    GROUP BY 1
)
SELECT
    year, total_ev_sales,
    LAG(total_ev_sales) OVER (ORDER BY year)   AS prev_year_ev_sales,
    ROUND(
        (total_ev_sales - LAG(total_ev_sales) OVER (ORDER BY year))::DECIMAL
        / NULLIF(LAG(total_ev_sales) OVER (ORDER BY year), 0) * 100, 1
    )                                          AS yoy_growth_pct
FROM ev_sales;

💡 Business value: Quantifies Mahindra's EV transition speed — a direct boardroom metric for strategy and investment.


📊 Power BI Executive Dashboard
Key KPIs
KPIValueBusiness SignalTotal Revenue₹19.36 BillionFull fleet performance baselineTotal Orders15,000Overall demand volumeDelivered Orders12,000Operational fulfillment countDelivery Rate79.8%Fulfillment health indicatorCancellation Rate10.5%Revenue leakage — ~₹2bn at riskAverage Order Value₹1.29 MillionPer-vehicle revenue benchmarkEV Revenue Share29.6%Electric transition momentumDiesel Revenue Share55.9%ICE portfolio still dominant
DAX Measures
daxDelivery Rate % =
DIVIDE(
    CALCULATE(COUNTROWS(orders), orders[order_status] = "Delivered"),
    COUNTROWS(orders)
)

Cancellation Rate % =
DIVIDE(
    CALCULATE(COUNTROWS(orders), orders[order_status] = "Cancelled"),
    COUNTROWS(orders)
)

EV Revenue % =
DIVIDE(
    CALCULATE(SUM(orders[final_price]), vehicles[fuel_type] = "Electric"),
    SUM(orders[final_price])
)

Diesel Revenue % =
DIVIDE(
    CALCULATE(SUM(orders[final_price]), vehicles[fuel_type] = "Diesel"),
    SUM(orders[final_price])
)

Petrol Revenue % =
DIVIDE(
    CALCULATE(SUM(orders[final_price]), vehicles[fuel_type] = "Petrol"),
    SUM(orders[final_price])
)
Dashboard Features

✅ Executive KPI cards — color-coded green (healthy) / red (alert)
✅ Monthly revenue trend with year slicer (2023–2026)
✅ Revenue by vehicle category — EV leads at ~₹6bn
✅ Top vehicle models by revenue — XEV 9e, BE 6, XUV700
✅ Order status breakdown — Delivered / Cancelled / Pending
✅ Fuel-type segmentation — Diesel 55.9% / EV 29.6% / Petrol 14.5%
✅ Interactive slicers for fuel type and year


🔍 Key Business Insights
📌  EV is the #1 revenue category at ~₹6bn — Mahindra's electric
    portfolio is the largest revenue driver by category, ahead of SUVs.

📌  Diesel still dominates fuel-type revenue at 55.9%, but EV at
    29.6% signals rapid transition. The crossover is a key metric
    to track over the next 2–3 years.

📌  10.5% cancellation rate = ~1,575 lost orders. At ₹1.29M average
    order value, this represents ~₹2bn in annual revenue leakage.

📌  XEV 9e leads all models at ~₹3bn, followed by BE 6 — both new
    EV launches confirming strong product-market fit for Mahindra Electric.

📌  Cohort analysis identifies which acquisition months generated the
    most loyal customers — directly informs campaign budget allocation.

📌  Pareto analysis confirms 80/20 rule holds — a small model portfolio
    drives the majority of revenue, guiding production prioritization.

📈 Analytics Coverage
ModuleFileQueriesKey ConceptsSalesSALES_ANALYTICS.sql5CTE, LAG, DENSE_RANK, multi-JOINCustomerCUSTOMER_ANALYTICS.sql5CLV, DENSE_RANK, HAVING, 3-table JOINDealershipDEALERSHIP_ANALYTICS.sql4ROW_NUMBER, date arithmetic, CASE WHENFinanceFINANCE_ANALYTICS.sql4Conditional aggregation, ratio analysisServiceSERVICE_ANALYTICS.sql4DENSE_RANK, retention subqueryAdvancedADVANCED_SQL_ANALYTICS.sql6PARTITION BY, AGE(), SUM OVER, NULLIF, ParetoTotal6 files28Window functions, CTEs, cohorts, running totals

🛠️ Tech Stack
ToolPurposePostgreSQL 16Primary database and query engineSQL28 business analytics queries across 6 modulesPower BI DesktopExecutive KPI dashboardDAXCalculated measures for KPI reportingPower QueryData transformation and ETLGitHubVersion control and portfolio hosting

🚀 How to Run
1. Clone the repository
bashgit clone https://github.com/saptakbasu07/mahindra-automotive-business-analytics.git
cd mahindra-automotive-business-analytics
2. Set up PostgreSQL
sqlCREATE DATABASE mahindra_analytics;
\c mahindra_analytics

\COPY customers   FROM 'data/customers.csv'   CSV HEADER;
\COPY vehicles    FROM 'data/vehicles.csv'    CSV HEADER;
\COPY orders      FROM 'data/orders.csv'      CSV HEADER;
\COPY dealerships FROM 'data/dealerships.csv' CSV HEADER;
3. Run SQL modules
bashpsql -d mahindra_analytics -f sql/SALES_ANALYTICS.sql
psql -d mahindra_analytics -f sql/CUSTOMER_ANALYTICS.sql
psql -d mahindra_analytics -f sql/DEALERSHIP_ANALYTICS.sql
psql -d mahindra_analytics -f sql/FINANCE_ANALYTICS.sql
psql -d mahindra_analytics -f sql/SERVICE_ANALYTICS.sql
psql -d mahindra_analytics -f sql/ADVANCED_SQL_ANALYTICS.sql
4. Open the Power BI dashboard
1. Open Power BI Desktop
2. File → Open → powerbi/Mahindra_Executive_Dashboard.pbix
3. Update data source path if prompted
4. Click Refresh All

👤 Author
Saptak Basu — BCA Student | Aspiring Data Analyst

