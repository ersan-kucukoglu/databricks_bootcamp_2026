# Bike Sales Data Lakehouse

A **Medallion Architecture** (Bronze-Silver-Gold) data lakehouse built on Databricks, integrating data from two source systems (CRM & ERP) to create a dimensional model for analytics.

📐 **[View Architecture Diagrams](ARCHITECTURE.md)** - Interactive visual diagrams showing data flow, technologies, and orchestration

---

## 📊 Architecture: 3-Layer Medallion Pattern

### Bronze Layer (Raw Data Ingestion)
* **Location**: `workspace.bronze` schema
* **6 tables** from two source systems:
  * **CRM System**: `crm_cust_info`, `crm_prd_info`, `crm_sales_details`
  * **ERP System**: `erp_cust_az12`, `erp_loc_a101`, `erp_px_cat_g1v2`
* **Process**: Raw CSV files ingested from `/Volumes` → Delta tables
* **Notebook**: `bronze_layer`

### Silver Layer (Cleaned & Transformed)
* **Location**: `workspace.silver` schema
* **6 cleaned tables**:
  * **CRM**: `crm_customers`, `crm_products`, `crm_sales`
  * **ERP**: `erp_customers`, `erp_customer_location`, `erp_product_category`
* **Transformations Applied**:
  * **Data Cleaning**: Trimmed whitespace, normalized codes (marital status, gender)
  * **Date Conversion**: Integer dates (YYYYMMDD) → DateType with validation
  * **NULL Handling**: Defensive logic for missing/zero values (e.g., `coalesce` for costs)
  * **Schema Standardization**: Consistent naming and types
* **Notebooks**: 
  * 3 CRM notebooks (one per entity)
  * 3 ERP notebooks (one per entity)
  * `silver_orchestration` (runs all silver transformations)

### Gold Layer (Business-Ready Dimensional Model)
* **Location**: `workspace.gold` schema
* **Star Schema** with 3 tables:
  * **Dimensions**: `dim_customers`, `dim_products`
  * **Fact**: `fact_sales`
* **Purpose**: Optimized for BI tools & analytics queries
* **Notebooks**: 
  * `gold_dim_customers`, `gold_dim_products`, `gold_fact_sales`
  * `gold_orchestration` (runs all gold transformations)

---

## ⚙️ Orchestration: Databricks Job

**Job Name**: `loading_bike_data_lakehouse`

**Pipeline**:
```
bronze_layer → silver_layer → gold_layer
```

* **3 sequential tasks** with dependency management
* **Schedule**: Weekly (Sundays at 10:17 AM Europe/Berlin)
* **Compute**: Performance-optimized serverless compute

---

## 🛠️ Technical Skills Demonstrated

1. **PySpark**: DataFrame transformations, data cleaning, type conversions
2. **SQL**: Schema management, Unity Catalog operations
3. **Delta Lake**: ACID transactions, table management
4. **Unity Catalog**: 3-tier namespace (catalog.schema.table)
5. **Medallion Architecture**: Industry-standard data lakehouse pattern
6. **Data Quality**: Defensive coding (null handling, validation checks)
7. **ETL/ELT**: Extract-Load-Transform with orchestration
8. **Version Control**: Git integration
9. **Dimensional Modeling**: Star schema design

---

## 📁 Code Organization

```
databricks_bootcamp_2026_de/
└── bike_lakehouse/
    ├── bronze/
    │   └── bronze_layer.ipynb
    ├── silver/
    │   ├── crm/
    │   │   ├── silver_crm_cust_info.ipynb
    │   │   ├── silver_crm_prd_info.ipynb
    │   │   └── silver_crm_sales_details.ipynb
    │   ├── erp/
    │   │   ├── silver_erp_cust_az12.ipynb
    │   │   ├── silver_erp_loc_a101.ipynb
    │   │   └── silver_erp_px_cat_g1v2.ipynb
    │   └── silver_orchestration.ipynb
    └── gold/
        ├── gold_dim_customers.ipynb
        ├── gold_dim_products.ipynb
        ├── gold_fact_sales.ipynb
        └── gold_orchestration.ipynb
```

---

## 🚀 Data Flow

1. **Bronze**: CSV files → Raw Delta tables (no transformations)
2. **Silver**: Data quality, cleaning, normalization, type conversions
3. **Gold**: Dimensional model (star schema) for analytics
4. **Orchestration**: Automated weekly execution via Databricks Job

---

## 📋 Key Transformations

### Data Quality Patterns
* Trimming whitespace from all string columns
* Normalizing categorical values (S→Single, M→Married, F→Female, M→Male)
* Defensive date parsing with validation (zero values → NULL)
* NULL handling for numeric fields using `coalesce`
* Schema validation and type casting

### Date Handling Example
```python
# Convert integer dates (20101229) to DateType with validation
df = df.withColumn(
    "sls_order_dt",
    when(
        (col("sls_order_dt") == 0) | (length(col("sls_order_dt").cast("string")) != 8),
        None
    ).otherwise(to_date(col("sls_order_dt").cast("string"), "yyyyMMdd"))
)
```

---

## 📊 Data Sources

### CRM System
* **Customer Info**: Customer demographics, contact details
* **Product Info**: Product catalog with pricing and lifecycle
* **Sales Details**: Transactional sales data

### ERP System
* **Customer Master**: Additional customer attributes
* **Location Data**: Geographic information
* **Product Categories**: Product hierarchy and classifications

---

## 🎯 Business Value

* **Single Source of Truth**: Integrated view of sales data across CRM and ERP
* **Data Quality**: Consistent, validated, and clean data
* **Analytics-Ready**: Star schema optimized for BI tools
* **Automated**: Weekly refresh ensures up-to-date insights
* **Scalable**: Delta Lake architecture supports growing data volumes