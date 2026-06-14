# Architecture Diagram

## Data Flow Overview

```mermaid
flowchart TB
    subgraph Sources["📁 Data Sources"]
        CRM[("CRM System<br/>CSV Files")]
        ERP[("ERP System<br/>CSV Files")]
    end

    subgraph Bronze["🥉 Bronze Layer - Raw Data Ingestion"]
        direction TB
        B_Tech["Technologies:<br/>• PySpark<br/>• Delta Lake<br/>• Unity Catalog<br/>• CSV Reader"]
        B1["crm_cust_info"]
        B2["crm_prd_info"]
        B3["crm_sales_details"]
        B4["erp_cust_az12"]
        B5["erp_loc_a101"]
        B6["erp_px_cat_g1v2"]
    end

    subgraph Silver["🥈 Silver Layer - Cleaned & Transformed"]
        direction TB
        S_Tech["Technologies:<br/>• PySpark DataFrames<br/>• Data Quality Functions<br/>• Type Conversions<br/>• Validation Logic"]
        S_Transform["Transformations:<br/>• Trim whitespace<br/>• Normalize codes<br/>• Date conversion<br/>• NULL handling<br/>• Schema standardization"]
        S1["crm_customers"]
        S2["crm_products"]
        S3["crm_sales"]
        S4["erp_customers"]
        S5["erp_customer_location"]
        S6["erp_product_category"]
    end

    subgraph Gold["🥇 Gold Layer - Dimensional Model"]
        direction TB
        G_Tech["Technologies:<br/>• Star Schema Design<br/>• SQL Joins<br/>• Aggregations<br/>• Business Logic"]
        DIM1["dim_customers<br/>(Dimension)"]
        DIM2["dim_products<br/>(Dimension)"]
        FACT["fact_sales<br/>(Fact Table)"]
    end

    subgraph Orchestration["⚙️ Orchestration"]
        JOB["Databricks Job<br/>loading_bike_data_lakehouse<br/>Schedule: Weekly (Sun 10:17 AM)"]
        TASK1["Task 1:<br/>bronze_layer"]
        TASK2["Task 2:<br/>silver_orchestration"]
        TASK3["Task 3:<br/>gold_orchestration"]
    end

    subgraph Analytics["📊 Analytics & BI"]
        BI["Business Intelligence Tools<br/>Dashboards & Reports"]
    end

    CRM --> B1 & B2 & B3
    ERP --> B4 & B5 & B6
    
    B1 --> S1
    B2 --> S2
    B3 --> S3
    B4 --> S4
    B5 --> S5
    B6 --> S6
    
    S1 & S4 --> DIM1
    S2 & S6 --> DIM2
    S3 --> FACT
    
    DIM1 & DIM2 & FACT --> BI
    
    JOB --> TASK1
    TASK1 --> TASK2
    TASK2 --> TASK3
    
    TASK1 -.controls.-> Bronze
    TASK2 -.controls.-> Silver
    TASK3 -.controls.-> Gold

    style Sources fill:#e1f5ff
    style Bronze fill:#cd7f32,color:#fff
    style Silver fill:#c0c0c0
    style Gold fill:#ffd700
    style Orchestration fill:#90EE90
    style Analytics fill:#DDA0DD
```

---

## Layer Details

### 🥉 Bronze Layer (Raw Zone)
**Purpose**: Ingest raw data with no transformations

**Technologies**:
* PySpark for data reading
* Delta Lake for ACID transactions
* Unity Catalog for governance
* CSV file format support

**Schema**: `workspace.bronze`

**Tables**: 6 raw tables (3 CRM + 3 ERP)

---

### 🥈 Silver Layer (Cleansed Zone)
**Purpose**: Clean, validate, and standardize data

**Technologies**:
* PySpark DataFrame transformations
* Built-in data quality functions (trim, coalesce, when)
* Type casting and conversions
* Custom validation logic

**Key Transformations**:
1. **String Cleaning**: Remove leading/trailing spaces
2. **Code Normalization**: S→Single, M→Married/Male, F→Female
3. **Date Conversion**: YYYYMMDD integers → DateType with validation
4. **NULL Handling**: Defensive coding with coalesce and validation
5. **Schema Standardization**: Consistent column names and types

**Schema**: `workspace.silver`

**Tables**: 6 cleaned tables

---

### 🥇 Gold Layer (Curated Zone)
**Purpose**: Business-ready dimensional model for analytics

**Technologies**:
* Star schema design
* SQL for complex joins
* Business logic implementation
* Performance optimization

**Model**: Star Schema
* **2 Dimensions**: Customer & Product
* **1 Fact Table**: Sales transactions

**Schema**: `workspace.gold`

**Tables**: 3 tables (2 dimensions + 1 fact)

---

## Technology Stack

```mermaid
graph LR
    A["Databricks"] --> B["PySpark"]
    A --> C["Delta Lake"]
    A --> D["Unity Catalog"]
    A --> E["Databricks Jobs"]
    A --> F["Notebooks"]
    
    B --> G["Data Transformations"]
    C --> H["ACID Transactions"]
    D --> I["Data Governance"]
    E --> J["Orchestration"]
    F --> K["Code Organization"]
    
    style A fill:#FF3621,color:#fff
    style B fill:#E25A1C,color:#fff
    style C fill:#00ADD4,color:#fff
    style D fill:#1B3139,color:#fff
    style E fill:#90EE90
    style F fill:#DDA0DD
```

---

## Orchestration Flow

```mermaid
sequenceDiagram
    participant Job as Databricks Job
    participant Bronze as Bronze Layer
    participant Silver as Silver Layer
    participant Gold as Gold Layer
    participant Catalog as Unity Catalog
    
    Note over Job: Weekly Schedule Trigger<br/>(Sundays 10:17 AM)
    
    Job->>Bronze: Task 1: Execute bronze_layer
    Bronze->>Catalog: Read CSV from /Volumes
    Bronze->>Catalog: Write to workspace.bronze.*
    Bronze-->>Job: ✓ Task 1 Complete
    
    Job->>Silver: Task 2: Execute silver_orchestration
    Silver->>Catalog: Read from workspace.bronze.*
    Note over Silver: Apply transformations:<br/>clean, validate, convert
    Silver->>Catalog: Write to workspace.silver.*
    Silver-->>Job: ✓ Task 2 Complete
    
    Job->>Gold: Task 3: Execute gold_orchestration
    Gold->>Catalog: Read from workspace.silver.*
    Note over Gold: Build dimensional model:<br/>star schema
    Gold->>Catalog: Write to workspace.gold.*
    Gold-->>Job: ✓ Task 3 Complete
    
    Note over Job: Pipeline Complete ✓
```

---

## Data Quality Framework

```mermaid
flowchart LR
    Input["Raw Data"] --> Val1{"Zero/NULL<br/>Check"}
    Val1 -->|Invalid| Reject["Set to NULL"]
    Val1 -->|Valid| Val2{"Format<br/>Validation"}
    Val2 -->|Invalid| Reject
    Val2 -->|Valid| Transform["Apply<br/>Transformation"]
    Transform --> Val3{"Type<br/>Validation"}
    Val3 -->|Invalid| Reject
    Val3 -->|Valid| Output["Clean Data"]
    
    style Input fill:#e1f5ff
    style Output fill:#90EE90
    style Reject fill:#ffcccc
    style Val1 fill:#fff4e6
    style Val2 fill:#fff4e6
    style Val3 fill:#fff4e6
    style Transform fill:#e6f3ff
```