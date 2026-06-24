# sql-bikestore-analysis
SQL Server Bike Store Analysis Project with KPIs, Views, Stored Procedures, and SQL Server Agent Jobs
[README_1.md](https://github.com/user-attachments/files/29291398/README_1.md)
# 🚲 BikeStore — SQL-Powered Retail Performance System

A complete SQL Server analytics project built on a multi-store bike retail dataset.
The project covers everything from database design and data loading to KPI reporting,
automated jobs, and customer-level profiling.

---

## 📁 Project Structure

```
BikeStore/
├── sql_bikestore_project.sql          # Full SQL script (DDL + DML + Views + SPs + KPIs)
├── querry results screenshots/        # Query output screenshots
│   ├── company's total revenue KPI.png
│   ├── average order value KPI.png
│   ├── revenue_by_store_KPI.png
│   ├── gross_profit_per_category_KPI.png
│   ├── sales_by_brand_KPI.png
│   ├── staff_revenue_KPI.png
│   ├── items_moving_slow_KPI.png
│   ├── vw_storesalessummary.png
│   ├── vw_regionaltrends.png
│   ├── vw_SalesByCategory.png
│   ├── vw_inventorystatus.png
│   ├── calcukate store KPI stored proc.png
│   ├── compare sales over year Stored proc.png
│   └── get_custimer profile stored proc.png
└── sql server agent job setup/        # Automated job screenshots
    ├── sql_server agent_1.png
    ├── server agent_ steps_2.png
    └── sql_server_agent_schedules .png
```

---

## 🗄️ Database Schema

The database consists of **9 normalized tables** connected via primary and foreign keys.


| Table | Description |
|---|---|
| `customers` | Customer personal info (name, email, city, state) |
| `orders` | Order header — status, dates, linked to customer & store |
| `order_items` | Order line items — product, quantity, price, discount |
| `products` | Product catalog — brand, category, model year, list price |
| `categories` | Product categories (Mountain Bikes, Road Bikes, etc.) |
| `brands` | Brand master list (Trek, Electra, Surly, etc.) |
| `stores` | Store locations (Baldwin Bikes, Santa Cruz Bikes, Rowlett Bikes) |
| `staffs` | Staff info — linked to store, includes manager hierarchy |
| `stocks` | Inventory levels per product per store |

---

## 📊 Views

Six analytical views were created to support reporting without repeating complex queries.

### `vw_StoreSalesSummary`
Aggregates total orders, total revenue, and average order value per store.


### `vw_TopSellingProducts`
Ranks products by total units sold and total revenue using `DENSE_RANK()`.

### `vw_InventoryStatus`
Labels each product-store combination as `sufficient`, `low`, `critical`, or `out of stock` based on stock quantity.

### `vw_StaffPerformance`
Shows total orders handled and revenue generated per staff member.

### `vw_RegionalTrends`
Breaks down total orders and revenue by city and state.

### `vw_SalesByCategory`
Sales volume, total revenue, and discount rate per product category.

---

## ⚙️ Stored Procedures

### `sp_CalculateStoreKPI` — `@store_id`
Returns a full KPI report for a single store in one call:
- Store info (name, city, state)
- Total orders, revenue, AOV, units sold
- Top 5 best-selling products
- Top performing staff
- Out-of-stock products

```sql
EXEC sp_CalculateStoreKPI @store_id = 1;
```
---

### `sp_GenerateRestockList` — `@store_id`, `@threshold` (optional, default 10)
Lists all products at or below the stock threshold for a given store — ready to send to procurement.

```sql
EXEC sp_GenerateRestockList @store_id = 1;
EXEC sp_GenerateRestockList @store_id = 1, @threshold = 10;
```

---

### `sp_CompareSalesYearOverYear` — `@year1`, `@year2`
Side-by-side comparison of two years: total orders, units sold, revenue, AOV, and percentage growth/decline.

```sql
EXEC sp_CompareSalesYearOverYear @year1 = 2017, @year2 = 2018;
```
> **Key finding:** Revenue dropped **-47.36%** from 2017 to 2018 (3,447 → 1,814 orders).

---

### `sp_GetCustomerProfile` — `@customer_id`
Full 360° customer view:
- Personal info (name, email, city)
- Overall spending summary (total orders, total spend, AOV, first/last order date)
- Top 5 most purchased products

```sql
EXEC sp_GetCustomerProfile @customer_id = 1;
```
---

## 📈 Business KPIs

### 1. Company Total Revenue

```sql
SELECT SUM(total_revenue) AS company_total_revenue
FROM vw_StoreSalesSummary;
```

> **$7,689,116.56** across all stores and years.

---

### 2. Average Order Value (AOV) by Store

```sql
SELECT store_name, avg_order_value
FROM vw_StoreSalesSummary;
```
| Store | AOV |
|---|---|
| Rowlett Bikes | $4,985.87 |
| Santa Cruz Bikes | $4,614.43 |
| Baldwin Bikes | $4,771.96 |

---

### 3. Revenue by Store

```sql
SELECT store_name, total_revenue, total_orders
FROM vw_StoreSalesSummary
ORDER BY total_revenue DESC;
```
| Store | Revenue | Orders |
|---|---|---|
| Baldwin Bikes | $5,215,751.28 | 1,093 |
| Santa Cruz Bikes | $1,605,823.04 | 348 |
| Rowlett Bikes | $867,542.24 | 174 |

---

### 4. Gross Profit by Category

```sql
SELECT category_name, total_revenue, total_discount_given, discount_rate_pct
FROM vw_SalesByCategory
ORDER BY total_revenue DESC;
```
> **Mountain Bikes** leads with **$2,715,079** in revenue.

---

### 5. Sales by Brand

```sql
SELECT * FROM vw_SalesByBrand
ORDER BY brand_id;
```
> **Trek** is the top brand: **$4,602,754** revenue, **1,839** units sold.

---

### 6. Staff Revenue Contribution

```sql
SELECT staff_name, total_orders, total_revenue
FROM vw_StaffPerformance
ORDER BY total_revenue DESC;
```
> **Marcelene Boyer** leads: **553 orders**, **$2,624,120** in revenue.

---

### 7. Slow-Moving Inventory

```sql
SELECT store_name, product_name, stock_quantity, stock_status
FROM vw_InventoryStatus
ORDER BY stock_quantity DESC;
```
---

## 🤖 SQL Server Agent — Automated Daily Job

A SQL Server Agent job (`BikeStore_Daily_Job`) was configured to run automatically on a daily schedule, keeping all views and KPI data up to date without manual intervention.

## 🛠️ Technologies Used

- **SQL Server 2022** (SSMS 16.0)
- **T-SQL** — DDL, DML, Views, Stored Procedures, Window Functions
- **SQL Server Agent** — Scheduled automation
- **BULK INSERT** — CSV data loading

---

## 🔑 Key Findings

| Insight | Value |
|---|---|
| Total company revenue | $7,689,116.56 |
| Best performing store | Baldwin Bikes ($5.2M) |
| Top product category | Mountain Bikes ($2.7M) |
| Top brand | Trek ($4.6M, 1,839 units) |
| Top staff member | Marcelene Boyer ($2.6M) |
| YoY revenue change (2017→2018) | **-47.36%** |
| Highest AOV store | Rowlett Bikes ($4,985.87) |

---

## 👤 Author

**Ibrohimov Mansur**
