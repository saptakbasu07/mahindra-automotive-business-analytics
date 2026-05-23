# 🚗 Mahindra Automotive SQL Business Analytics

A comprehensive business analytics project simulating real-world Mahindra Automotive operations using PostgreSQL and advanced SQL techniques.

---

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Dataset](#dataset)
- [SQL Concepts](#sql-concepts-used)
- [Business Problems](#business-problems-solved)
- [Key Insights](#key-insights)
- [Project Structure](#project-structure)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [Usage](#usage)
- [Queries & Results](#queries--results)
- [Author](#author)

---

## 🎯 Project Overview

This project demonstrates enterprise-level SQL analytics by analyzing Mahindra Automotive's operations across multiple business domains:

- **Sales Analytics** - Revenue trends, model performance, and market segmentation
- **Customer Analytics** - Behavior patterns, retention rates, and customer lifecycle
- **Dealer Performance** - Delivery efficiency, sales volume, and operational metrics
- **Financing Analysis** - Loan dependency by vehicle category and payment patterns
- **EV Growth Trends** - Electric vehicle adoption and market share growth
- **Delivery Delay Analysis** - Logistics performance and SLA compliance
- **Service Analytics** - Maintenance patterns, service frequency, and customer satisfaction

---

## 📊 Dataset

The project includes normalized tables representing real-world automotive operations:

| Table | Description |
|-------|-------------|
| **Customers** | Customer demographics, contact info, and registration date |
| **Orders** | Sales transactions with order status and pricing |
| **Vehicles** | Vehicle models, categories (SUV, Sedan, Hatchback), specs |
| **Dealerships** | Dealer locations, regions, and performance metrics |
| **Financing** | Loan records, terms, interest rates, and approval status |
| **Service Records** | Service dates, issues, costs, and technician info |
| **Test Drives** | Test drive bookings and customer conversions |

---

## 🛠️ SQL Concepts Used

- **JOINs** - INNER, LEFT, RIGHT, FULL OUTER joins for multi-table analysis
- **CTEs (Common Table Expressions)** - Recursive and non-recursive queries for complex logic
- **Window Functions** - ROW_NUMBER(), RANK(), LAG(), LEAD() for advanced analytics
- **Aggregations** - GROUP BY, HAVING, multiple aggregation functions
- **Ranking Functions** - Top-N queries and percentile analysis
- **Cohort Analysis** - Customer lifecycle and retention tracking
- **Retention Analysis** - Repeat purchase patterns and churn prediction
- **Time-Series Analytics** - Trend analysis, moving averages, and forecasting

---

## 💡 Business Problems Solved

1. **Which vehicle models generate the highest revenue?**
   - Revenue ranking by model with delivery status filtering

2. **Which dealers have the highest delivery delays?**
   - SLA breach analysis and dealer performance comparison

3. **Which states contribute the highest sales?**
   - Geographic revenue distribution and market penetration

4. **Which customers are repeat buyers?**
   - Customer segmentation and loyalty analysis

5. **Which vehicle categories depend most on financing?**
   - Financing adoption rates by category and price range

6. **Which models require frequent servicing?**
   - Service frequency analysis and maintenance cost prediction

---

## 🔍 Key Insights

| Insight | Finding |
|---------|---------|
| **Top Revenue Generator** | SUVs generated the highest overall revenue |
| **EV Market Growth** | Electric vehicle sales showed strong yearly growth trends |
| **Delivery Performance** | Some dealerships exceeded 30-day average delivery delays |
| **Financing Trends** | Premium vehicles showed 65%+ financing dependency |
| **Customer Retention** | Strong retention through service engagement programs |

---

## 📁 Project Structure

```
mahindra-automotive-business-analytics/
├── data/                          # Dataset files
│   ├── customers.csv
│   ├── orders.csv
│   ├── vehicles.csv
│   ├── dealerships.csv
│   ├── financing.csv
│   ├── service_records.csv
│   └── test_drives.csv
│
├── sql/                           # SQL query files
│   ├── 01_schema_setup.sql        # Database and table creation
│   ├── 02_data_import.sql         # Data loading scripts
│   ├── 03_sales_analytics.sql     # Revenue and sales queries
│   ├── 04_customer_analytics.sql  # Customer behavior queries
│   ├── 05_dealer_performance.sql  # Dealer metrics
│   ├── 06_financing_analysis.sql  # Financing patterns
│   └── 07_advanced_queries.sql    # Complex analytics queries
│
├── screenshots/                   # Query results and visualizations
│
├── README.md                      # Project documentation
└── .gitignore
```

---

## 💻 Tech Stack

| Component | Technology |
|-----------|-----------|
| **Database** | PostgreSQL 12+ |
| **Query Language** | SQL (Standard + PostgreSQL Extensions) |
| **Version Control** | Git & GitHub |
| **Tool** | pgAdmin / DBeaver / CLI |

---

## 🚀 Getting Started

### Prerequisites

- PostgreSQL 12 or higher installed
- A SQL IDE (pgAdmin, DBeaver, or psql CLI)
- Git for version control

### Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/saptakbasu07/mahindra-automotive-business-analytics.git
   cd mahindra-automotive-business-analytics
   ```

2. **Create a new database:**
   ```sql
   CREATE DATABASE mahindra_analytics;
   ```

3. **Run the schema setup:**
   ```bash
   psql -U postgres -d mahindra_analytics -f sql/01_schema_setup.sql
   ```

4. **Import the data:**
   ```bash
   psql -U postgres -d mahindra_analytics -f sql/02_data_import.sql
   ```

---

## 📖 Usage

### Running Queries

Execute individual analysis queries:

```bash
psql -U postgres -d mahindra_analytics -f sql/03_sales_analytics.sql
```

Or connect interactively and paste queries:

```bash
psql -U postgres -d mahindra_analytics
```

### Example: Finding Top Revenue-Generating Vehicles

```sql
WITH revenue AS (
    SELECT 
        vehicle_id,
        SUM(final_price) AS total_revenue,
        COUNT(*) AS units_sold
    FROM orders
    WHERE order_status = 'Delivered'
    GROUP BY vehicle_id
)

SELECT 
    v.model_name,
    v.category,
    r.total_revenue,
    r.units_sold,
    ROUND(r.total_revenue / r.units_sold::NUMERIC, 2) AS avg_price
FROM revenue r
JOIN vehicles v ON r.vehicle_id = v.vehicle_id
ORDER BY total_revenue DESC
LIMIT 10;
```

---

## 📊 Queries & Results

Key analytical queries included in the project:

- **Sales Performance**: Top models, seasonal trends, revenue forecasting
- **Customer Insights**: Segmentation, lifetime value, churn prediction
- **Operational Metrics**: Dealer efficiency, delivery times, inventory management
- **Financial Analysis**: Financing penetration, payment defaults, profitability
- **Trend Analysis**: YoY growth, market share, competitive positioning

See the `sql/` directory for complete query implementations.

---

## 🎓 Learning Outcomes

This project demonstrates:

✅ Enterprise SQL design and optimization  
✅ Business analytics problem-solving  
✅ Real-world data modeling  
✅ Complex query development  
✅ Performance optimization techniques  
✅ Documentation best practices  

---

## 📝 License

This project is open source and available for educational purposes.

---

## 👨‍💻 Author

**Saptak Basu**

- GitHub: [@saptakbasu07](https://github.com/saptakbasu07)
- Project: [Mahindra Automotive Business Analytics](https://github.com/saptakbasu07/mahindra-automotive-business-analytics)

---

## 📞 Support & Contributions

Have suggestions or found an issue? Feel free to:
- Open a GitHub issue
- Submit a pull request
- Contact the author

---

**Last Updated**: May 2026  
**Status**: Active & Maintained
