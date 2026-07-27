# Data Warehouse Architectures

How we physically build and populate our warehouse defines its architecture. There are three classic approaches, plus the modern twist:

---

## A. Inmon's Top-Down

**Concept:** Build the massive, normalized, enterprise-wide warehouse first (the "Single Source of Truth"). Then, build smaller departmental data marts from this central warehouse. Think of it as building the main engine of a factory first, then attaching specific conveyor belts (marts) to it.

**Aquaculture Example:** We build a giant central warehouse with 500 tightly normalized tables (e.g., Farm, Pen, Fish_Batch, Feed_Order, Health_Event). Only after this is complete, we slice out a "Logistics Mart" for the harvest team. This is great for data consistency but takes 18 months to build.

**Pros:** Highly consistent, minimal data redundancy.

**Cons:** Very slow and expensive to build; rigid to change.

---
## B. Kimball's Bottom-Up

**Concept:** Build departmental data marts first (e.g., Sales Mart, Inventory Mart) directly from the source systems, using a dimensional model (star schemas). These marts are then joined together logically via a "bus architecture" to create a federated warehouse. Think of it as building separate kitchen stations (grill, pastry, salad) and connecting them with a common ordering system.

**Aquaculture Example:** The Feed Team builds their own Feed_Consumption_Mart directly from feed barge data. The Vet Team builds their Health_Mart directly from veterinary logs. Later, we use a common Date and Farm dimension to join these marts for enterprise reporting.

**Pros:** Fast to implement, business users get value quickly.

**Cons:** Data redundancy (same fish weight stored in 3 different marts), potential inconsistency if dimensions aren't standardized.

---

## C. Data Vault

**Concept:** A hybrid approach that separates data into three types: **Hubs** (core business keys, like Farm_ID), **Links** (relationships between hubs, like which feed is used at which farm), and **Satellites** (historical attributes, like changing water temperatures over time). It is designed for extreme scalability and full historical tracking.

**Aquaculture Example:** If a salmon farm changes its name or ownership, a Data Vault tracks every single change historically without overwriting old records. This is critical for regulatory compliance (traceability from egg to plate).

**Pros:** Handles change effortlessly, perfect for auditing, highly scalable.

**Cons:** Complex to query for business users; requires a separate "information mart" layer on top for reporting.

---

## D. Modern Cloud-Native

**Concept:** Used by Snowflake, Databricks, and BigQuery. Storage (cheap S3/ADLS) is completely separated from compute (query engines). We can scale up 100 servers to run a massive query, then scale down to zero to save costs.

**Aquaculture Example:** During the annual harvest season (September-October), we spin up massive compute clusters to run complex yield forecasts across 50 farms. In the slow winter months, we scale compute down to near-zero, saving 90% of cloud costs.

----

## Cubes (Multidimensional OLAP) or DATA CUBES

A data cube is not a physical 3D object; it is a multidimensional array of data that allows us to pre-aggregate measures across multiple business dimensions for lightning-fast queries.
A cube has:

| Component | Description |
|-----------|-------------|
| **Dimensions** | The "categories" we slice by (e.g., Farm Location, Date, Feed Type) |
| **Measures** | The "numbers" we analyze (e.g., Total Biomass, Average Daily Growth, Total Feed Consumed) |

---

### Aquaculture Example

Imagine a 3D cube (we can add more dimensions, but humans visualize 3):

| Axis | Dimension |
|------|-----------|
| **X** | Farm Location (Norway, Chile, Scotland) |
| **Y** | Date (Year → Quarter → Month → Day) |
| **Z** | Feed Type (Standard Pellet, High-Omega, Plant-Based) |

**The value inside the cube at the intersection** is **Total Feed Consumed (kg)**.

**Query:** *"Show me total feed consumed in Norway, in Q3 2025, for High-Omega feed."*

- **With Cube:** The cube grabs that single cell instantly — returns in **50 milliseconds**
- **Without Cube:** The warehouse would have to scan millions of transaction rows, filter, group, and sum — taking **30 seconds**

---

### Modern Reality

Physical cubes (Microsoft Analysis Services, Oracle Essbase) are less common today. Modern warehouses like Snowflake and BigQuery use adaptive query engines that build temporary aggregates on the fly, effectively acting as "virtual cubes."
