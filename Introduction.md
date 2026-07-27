## Data Lake

---

### Concept

A data lake stores raw, unprocessed data in its native format. In aquaculture, this is the chaotic firehose of data streaming in 24/7 from the farm. We store everything exactly as it arrives, because we don't know which data points will be valuable for AI models in the future.

---

### Processing

**ELT (Extract, Load, Transform):** We dump it raw, and clean it up only when we decide to use it.

---

### Scope

Enterprise-wide raw repository.

---

### Aquaculture Example

Our open-pen salmon farm has hundreds of underwater IoT sensors. Every second, the lake ingests:

- Raw video footage of feeding behavior (MP4 files)
- Unstructured JSON logs from oxygen/temperature probes
- Historical PDFs of veterinary health reports
- Acoustic sonar pings tracking fish biomass

---

### Major Providers

| Provider | Service |
|----------|---------|
| AWS | S3 + Lake Formation |
| Azure | Data Lake Storage (ADLS) |
| Google | Dataplex |

---
## Data Warehouse

---

### Concept

A data warehouse takes that raw ocean data, cleans it, standardizes it, and structures it into strict rows and columns (facts and dimensions). It is the **"Single Source of Truth"** for the entire aquaculture corporation. It answers historical, strategic business questions.

---

### Processing

**ETL (Extract, Transform, Load):** Data is heavily scrubbed, deduplicated, and modeled before it enters.

---

### Scope

Enterprise-wide, strategic decision-making.

---

### Aquaculture Example

Every night, an ETL pipeline pulls data from 50 different salmon farms, transforms it, and loads it into a central warehouse (e.g., Snowflake). It creates tables like:

- **Fact_Daily_Growth** — avg weight, temperature, feed conversion ratio
- **Dim_Farm_Sites** — location, water salinity, depth
- **Dim_Stock_Vaccinations** — dates, batch numbers

The Global Head of Production runs a query on this warehouse:

> *"Across all our Norwegian farms, what was the average Feed Conversion Ratio (FCR) during the warm summers of 2023 vs. 2024?"*

The warehouse processes this in seconds, helping the CEO decide whether to invest in warmer-water feed strains.

---

### Major Providers

 **Snowflake**: Huge in aquaculture for its easy data sharing with third-party weather forecasters, **Amazon Redshift**, **Google BigQuery**, **Microsoft Azure Synapse**

---
## Data Mart

---

### Concept

A data mart is a small, subject-specific slice of the warehouse, customized for a single department. It contains only the data that the team cares about, making their daily queries lightning-fast and simple.

---

### Processing

Built from the central warehouse (or directly from source systems).

---

### Scope

Departmental or team-specific (tactical, day-to-day).

---

### Aquaculture Example

The Harvest & Logistics Team does not care about vaccine history or breeding genetics. They pull a subset from the main warehouse to create their own **Harvest Data Mart**. This mart contains just three tables:

- **Scheduled_Harvest_Dates**
- **Current_Biomass_per_Pen**
- **Truck_Fleet_Locations**

The Logistics Manager queries this mart 20 times a day:

> *"Show me all pens within 50 km of the processing plant that are at exactly 5.5 kg average weight so I can schedule tomorrow's harvest barges."*

Because the mart ignores 90% of the enterprise data, this query runs instantly on a simple BI dashboard (like Power BI or Tableau), preventing costly overgrowth (fish getting too big for the processing machinery).

---

### Major Providers

Data marts are not sold separately — they are built inside the major platforms.

| Approach | Platform |
|----------|----------|
| Create a "Virtual Warehouse" | Snowflake (dedicated to Logistics) |
| Create a curated Schema | Databricks Unity Catalog |
| Materialize specific aggregated tables | dbt (data build tool) |

---

## Cubes (Multidimensional OLAP) or DATA CUBES

A data cube is not a physical 3D object; it is a multidimensional array of data that allows us to pre-aggregate measures across multiple business dimensions for lightning-fast queries.

---

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
