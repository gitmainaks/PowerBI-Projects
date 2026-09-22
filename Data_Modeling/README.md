

<div align="center">

# 🌌 Galaxy Schema Data Model

### A multi-fact, conformed-dimension analytical model for Power BI

*One source of truth. Many business processes. Zero ambiguity.*

<br/>

[![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)](https://powerbi.microsoft.com/)
[![DAX](https://img.shields.io/badge/DAX-Measures_&_RLS-0078D4?style=for-the-badge)](https://learn.microsoft.com/dax/)
[![Schema](https://img.shields.io/badge/Schema-Galaxy-6C3FC5?style=for-the-badge)](#-the-galaxy-schema)
[![Model](https://img.shields.io/badge/Model-OLAP-2E7D32?style=for-the-badge)](#-core-concepts)
[![Security](https://img.shields.io/badge/Security-Dynamic_RLS-C62828?style=for-the-badge&logo=shield&logoColor=white)](#-row-level-security-rls)
[![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](#-license)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen?style=for-the-badge&logo=github)](#-contributing)

<br/>

[📖 Overview](#-overview) •
[🌌 Schema](#-the-galaxy-schema) •
[🧱 Architecture](#-model-architecture) •
[🔗 Relationships](#-relationships) •
[🔐 Security](#-row-level-security-rls) •
[🧮 DAX](#-dax-examples)

</div>

---

## 📑 Table of Contents

- [📖 Overview](#-overview)
- [✨ Key Features](#-key-features)
- [🌌 The Galaxy Schema](#-the-galaxy-schema)
- [🧱 Model Architecture](#-model-architecture)
- [🔗 Relationships](#-relationships)
- [🧠 Core Concepts](#-core-concepts)
- [🚚 The Order-to-Cash Process](#-the-order-to-cash-process)
- [🧮 DAX](#-dax-examples)
- [🔐 Row-Level Security (RLS)](#-row-level-security-rls)
- [🏷️ Naming Standards](#️-naming-standards)
- [📄 License](#-license)

---

## 📖 Overview

This repository documents a **Galaxy Schema** (also called a *fact constellation*) built for **Power BI**. Instead of forcing every business process into a single fact table, the model keeps **multiple fact tables** — sales, order fulfilment, inventory, promotions, campaign spend, and sales targets — that all **share a common set of conformed dimensions** such as `dim_date`, `dim_product`, `dim_customer`, and `dim_campaign`.

Because the dimensions are shared, measures from different processes can be **safely compared side by side** in the same report — for example, *actual sales vs. target revenue*, or *campaign spend vs. promoted product sales*.

> 💡 **Why this matters:** A clean model is the foundation of fast, trustworthy reports. The right shape, the right grain, and the right relationships mean simpler DAX and fewer surprises.

---

## ✨ Key Features

| | Feature | Description |
|---|---|---|
| 🌌 | **Galaxy Schema** | Six fact tables sharing conformed dimensions |
| 📅 | **Shared Date Dimension** | One `dim_date` compares every process on a common timeline |
| 🎭 | **Role-Playing Dimension** | `dim_geo` serves both *bill-to* and *ship-to* cities via active / inactive relationships |
| 🗑️ | **Junk Dimension** | `dim_order_flags` bundles low-level order flags into one small table |
| 👻 | **Factless Fact Table** | `fact_promotion_coverage` tracks *whether* a promotion covered a product |
| 📸 | **Accumulating Snapshot** | `fact_order_process` follows an order through its lifecycle |
| 🔐 | **Dynamic RLS** | A `security` table filters rows by region using `USERPRINCIPALNAME()` |
| 📊 | **Central `_measures` Table** | All major measures live in one discoverable place |

---

## 🌌 The Galaxy Schema

<div align="center">

*Model view — multiple fact tables connected through shared, conformed dimensions.*

</div>

### 🗺️ Entity-Relationship View

<img width="879" height="470" alt="Galaxy Schema" src="https://github.com/user-attachments/assets/635d9b4c-1af4-45ce-a774-11709575b695" />

---

## 🧱 Model Architecture

Every data model is built from **three parts**: 🗂️ **Tables**, 🔗 **Relationships**, and 🧮 **Calculations**.

### 📊 Fact Tables — *"What happened?"*

| Table | Type | Grain (one row = …) | Connected to |
|---|---|---|---|
| `fact_sales` | Transactional | One sales order line (the *Details* of the business event) | `dim_customer`, `dim_product`, `dim_date`, `dim_order_flags`, `dim_geo` (×2) |
| `fact_order_process` | Accumulating snapshot | One order moving through its lifecycle | `dim_customer`, `dim_date` |
| `fact_inventory` | Periodic snapshot | One product per month | `dim_product`, `dim_date` |
| `fact_promotion_coverage` | Factless | One campaign–product pairing | `dim_campaign`, `dim_product` |
| `fact_campaign_spend` | Transactional | One campaign per date | `dim_campaign`, `dim_date` |
| `fact_sales_targets` | Target / budget | One target value per date | `dim_date` |

### 🧩 Dimension Tables — *"Describes what happened"*

| Table | Key | Role |
|---|---|---|
| `dim_date` | `date` | 📅 Shared (conformed) calendar used by five fact tables |
| `dim_customer` | `customer_id` | 👤 Customer attributes, including `region` (RLS target) |
| `dim_product` | `product_key` | 📦 Product attributes — shared by sales, inventory, and promotions |
| `dim_campaign` | `campaign_key` | 📣 Marketing campaign attributes — shared by spend and promotions |
| `dim_geo` | `geo_key` | 🌍 **Role-playing** dimension for bill-to and ship-to cities |
| `dim_order_flags` | `flag_key` | 🗑️ **Junk** dimension bundling small, unrelated flags |

### ⚙️ Supporting Tables

| Table | Purpose |
|---|---|
| `_measures` | 🧮 Holds all major DAX measures in one place |
| `security` | 🔐 Maps users to regions for dynamic Row-Level Security |

---

## 🔗 Relationships

| From (1) | To (\*) | Key | Notes |
|---|---|---|---|
| `dim_customer` | `fact_sales` | `customer_id` | One-to-many |
| `dim_customer` | `fact_order_process` | `customer_id` | One-to-many |
| `dim_product` | `fact_sales` | `product_key` | One-to-many |
| `dim_product` | `fact_inventory` | `product_key` | One-to-many |
| `dim_product` | `fact_promotion_coverage` | `product_key` | One-to-many |
| `dim_campaign` | `fact_promotion_coverage` | `campaign_key` | One-to-many |
| `dim_campaign` | `fact_campaign_spend` | `campaign_key` | One-to-many |
| `dim_order_flags` | `fact_sales` | `flag_key` | One-to-many |
| `dim_geo` | `fact_sales` | `bill_to_city_key` / `ship_to_city_key` | 🎭 Role-playing: one active, one inactive (dotted) |
| `dim_date` | `fact_sales` | `order_date` | One-to-many |
| `dim_date` | `fact_order_process` | `order_date` | One-to-many |
| `dim_date` | `fact_inventory` | `month` | One-to-many |
| `dim_date` | `fact_campaign_spend` | `date` | One-to-one, bi-directional |
| `dim_date` | `fact_sales_targets` | `date` | One-to-one, bi-directional |
| `security` | `dim_customer` | `region` | 🔐 RLS filter propagates from `security` to `dim_customer` |

> ⚠️ **Only one relationship between two tables can be active at a time.** Inactive relationships are activated per calculation with `USERELATIONSHIP()` — see [DAX](#-dax).

---

## 🧠 Core Concepts

### 🎭 Roles and Shapes

| Concept | Meaning |
|---|---|
| **Fact** | Records *what happened* — holds the numbers (measures) |
| **Dimension** | *Describes* what happened — who, what, where, when |
| **Star Schema** | One fact table surrounded by dimensions |
| **Snowflake Schema** | A large dimension is split into related tables; still a single fact table |
| **Galaxy Schema** ⭐ | **More than one fact table sharing the same dimensions** — the shape used here |

### 📐 Grain

> **Grain** is the level of detail that one row in a table represents.
> - A **dimension** row = *one customer*
> - A **fact** row = *one sale*

Knowing the grain first prevents double-counting and keeps relationships honest.

### 🧬 Special Dimension Patterns

| Pattern | Definition | Used in this model |
|---|---|---|
| 🤝 **Shared / Conformed Dimension** | One complete dimension that multiple facts connect to, so their measures can be compared safely | `dim_date`, `dim_product`, `dim_customer`, `dim_campaign` |
| 🎭 **Role-Playing Dimension** | One dimension joined to the same fact through several relationships so it plays different roles; only one is active at a time | `dim_geo` (bill-to vs. ship-to) |
| 🗑️ **Junk Dimension** | A small dimension that bundles several unrelated low-level flags | `dim_order_flags` |
| 🥸 **Dimension in Disguise** | A dimension extracted from the columns of a fact table | Applied when building `dim_*` tables from fact data |
| 👻 **Factless Fact** | A fact table of IDs and dates with **no measures** — it only tracks whether something happened | `fact_promotion_coverage` |
| 📸 **Accumulating Snapshot** | A fact table where one row tracks a process through several dated stages | `fact_order_process` |

### 🏛️ OLTP vs. OLAP

| | 🧾 OLTP (Transactional) | 📊 OLAP (This model) |
|---|---|---|
| **Goal** | Run the business, optimize the UI | Analyze the business |
| **Storage** | Same information stored repeatedly across tables | **Single point of truth** — stored once |
| **Shape** | Highly normalized | Fact + dimension tables |

### 🧾 Business Events: Header and Details

Transactions such as **orders, invoices, shipments, deliveries, production orders, bills of materials, and tickets** all have two parts:

| Section | Contains |
|---|---|
| **Header** | Address, customer, company, order number, order state, order date |
| **Details** | List of ordered products — quantity, price, final cost |

✅ **All numbers and core measures live in the Details.** Build the fact table from the Details (e.g. `fact_order_line`) and use the Header to derive dimension tables that connect back to it.

### 🔄 Unpivot

**Unpivot** turns columns into rows — a common Power Query step for reshaping wide source data into a tidy, fact-friendly format.

---

## 🚚 The Order-to-Cash Process

A delivery follows a chain of events. Each step is a dated fact, and together they form the **Accumulating Snapshot**:

```mermaid
flowchart LR
    A["🛒 Order<br/><small>Date · Number</small>"] --> B["📦 Ship<br/><small>Date</small>"]
    B --> C["🚚 Deliver<br/><small>Date</small>"]
    C --> D["🧾 Invoice<br/><small>Date · Number</small>"]
    D --> E["💳 Pay<br/><small>Date · Number</small>"]

    style A fill:#E3F2FD,stroke:#1565C0,color:#0D47A1
    style B fill:#EDE7F6,stroke:#5E35B1,color:#311B92
    style C fill:#E8F5E9,stroke:#2E7D32,color:#1B5E20
    style D fill:#FFF8E1,stroke:#F9A825,color:#5D4037
    style E fill:#FCE4EC,stroke:#C2185B,color:#880E4F
```

The **Order** is the main fact and the **single point of truth** for the order number. `dim_date` lets every stage be compared in the same view.

---

## 🧮 DAX

### 🎭 Activating an inactive relationship with `USERELATIONSHIP`

`USERELATIONSHIP` is a DAX function that turns on an inactive relationship **for one specific calculation**.

```dax
Sales by Bill-To City =
CALCULATE (
    [Sales Amount],
    USERELATIONSHIP ( fact_sales[bill_to_city_key], dim_geo[geo_key] )
)

Sales by Ship-To City =
CALCULATE (
    [Sales Amount],
    USERELATIONSHIP ( fact_sales[ship_to_city_key], dim_geo[geo_key] )
)
```

---

## 🔐 Row-Level Security (RLS)

This model uses **dynamic RLS**. The `security` table filters `dim_customer` by `region`, which in turn filters the related fact tables.

```dax
-- Role: Regional Access  |  Table filter on: security
[user_email] = USERPRINCIPALNAME ()
```

---

## 🏷️ Naming Standards

| Element | Convention | Example |
|---|---|---|
| Fact tables | `fact_` prefix | `fact_sales`, `fact_inventory` |
| Dimension tables | `dim_` prefix | `dim_product`, `dim_date` |
| Measure table | `_measures` (underscore keeps it at the top) | `_measures` |
| Surrogate keys | `_key` suffix | `product_key`, `campaign_key` |
| Business identifiers | `_id` suffix | `customer_id` |
| Lowercase, snake_case | Everywhere | `bill_to_city_key` |

---

## 📄 License

Distributed under the **MIT License**. See [`LICENSE`](LICENSE) for details.

---

<div align="center">

⭐ **If you found this project helpful, please consider giving it a star!** ⭐

<sub>Built with Power BI</sub>

</div>
