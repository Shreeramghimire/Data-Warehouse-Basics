## Data Lake

---

### Concept:

A data lake stores raw, unprocessed data in its native format. In aquaculture, this is the chaotic firehose of data streaming in 24/7 from the farm. We store everything exactly as it arrives, because we don't know which data points will be valuable for AI models in the future.

---

### Processing:

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

Microsoft Analysis Services and Oracle Essbase are less common today. Modern warehouses like Snowflake and BigQuery use adaptive query engines that build temporary aggregates on the fly, effectively acting as "virtual cubes."

----

## The Star Schema 

A central **Fact** table (e.g., Sales) surrounded by **Dimension** tables (e.g., Customer, Product, Date).

All the descriptive details about a product (name, brand, category, color) are crammed into a single, flat Product table.

**Visual:** It looks like a literal star.

### In Aquaculture 

In the Star Schema, all the descriptive details are crammed into single, wide dimension tables.

**The CAGE Dimension Table (One single table):**

| Cage_Key | Cage_Number | Farm_Site | Farm_Region | Water_Source | Stocking_Density | Pen_Type | Depth_Meters |
|----------|-------------|-----------|-------------|--------------|------------------|----------|--------------|
| 101 | Cage 7 | North Bay | Western Region | Fjord | 18 kg/m³ | Circular Steel | 25 |

**The FEED Dimension Table (One single table):**

| Feed_Key | Feed_Brand | Pellet_Size | Protein_Pct | Fat_Pct | Supplier | Supplier_Country |
|----------|------------|-------------|-------------|---------|----------|------------------|
| 500 | SuperGrow | 9mm | 45% | 22% | Skretting | Norway |

---

#### How It Looks

If we have 10,000 sensor readings from Cage 7, the words "North Bay," "Fjord," and "Circular Steel" are physically repeated in the database 10,000 times.

**The Query:** To find mortality in the Western Region, we just run:

```sql
SELECT SUM(Mortality) 
FROM FARM_OBSERVATIONS 
JOIN CAGE ON ... 
WHERE Farm_Region = 'Western Region';
```
----

## The Snowflake Schema 

The same central **Fact** table, but the surrounding dimensions are broken down into smaller, connected pieces.

Instead of one flat Product table, we have a Product table that links to a separate Brand table, which links to a separate Manufacturer table.

**Visual:** The edges of the star branch out into multiple levels, looking like a snowflake.

---

### In Aquaculture 

In the Snowflake Schema, we take that flat **CAGE** table and break it into smaller, logical pieces to remove repetition.

**The CAGE Dimension Table (Now tiny):**

| Cage_Key | Cage_Number | Farm_Site_Key | Stocking_Density | Pen_Type_Key | Depth_Meters |
|----------|-------------|---------------|------------------|--------------|--------------|
| 101 | Cage 7 | 5 | 18 kg/m³ | 12 | 25 |

**The FARM_SITE Table (New lookup):**

| Farm_Site_Key | Farm_Site | Water_Source | Region_Key |
|---------------|-----------|--------------|------------|
| 5 | North Bay | Fjord | 200 |

**The REGION Table (New lookup):**

| Region_Key | Farm_Region | Country |
|------------|-------------|---------|
| 200 | Western Region | Norway |

**The PEN_TYPE Table (New lookup):**

| Pen_Type_Key | Pen_Type | Manufacturer |
|--------------|----------|--------------|
| 12 | Circular Steel | Aqualine AS |

---

### How It Looks

The word "Western Region" is stored exactly once in the entire database. If we rename the region to "West Coast," we update just **1 row** in the REGION table, and it magically fixes every single sensor reading in history.

-----
## Star vs. Snowflake Schema: A Real Aquaculture Dilemma

Let's look at a real dilemma we will face in aquaculture:

---

### The Problem

Our feed changes constantly. Suppliers tweak protein percentages weekly, and pellet sizes vary by water temperature.

---

### The Star Schema Approach

In a Star Schema:

- Every time the feed changes by 0.5% protein, we have to overwrite thousands of historical rows in our fat FEED table
- This is a **maintenance nightmare** and we risk data corruption

**The Issue:** Historical data gets overwritten, making it impossible to analyze past performance against old feed formulations accurately.

---

### The Snowflake Schema Approach

In a Snowflake Schema:

- We create a `FEED_FORMULA` table linked to a `FEED_BATCH` table
- When the protein changes, we simply add a new `FEED_BATCH` key
- The old historical data stays **perfectly linked** to the old batch

**The Benefit:** We preserve historical accuracy while accommodating change — no overwriting, no corruption.

---

### Visual Comparison

| Aspect | Star Schema | Snowflake Schema |
|--------|-------------|------------------|
| **Feed changes** | Overwrite thousands of rows | Add one new batch row |
| **Historical accuracy** | Lost | Preserved |
| **Maintenance** | Nightmare | Clean and manageable |
| **Risk** | Data corruption | None |

---

### The Trade-Off

| Schema | Query Speed | Maintenance Ease | Historical Accuracy |
|--------|-------------|------------------|---------------------|
| **Star** |  Fast (1 join) |  Painful (overwrites) |  Lost |
| **Snowflake** |  Slower (3+ joins) |  Easy (append only) |  Preserved |

---

### Key Takeaway

> The Snowflake schema costs us a few extra joins at query time, and it saves us from a maintenance nightmare and preserves our historical data integrity, a critical requirement for aquaculture traceability and regulatory compliance.

### Choosing Between Star and Snowflake: A Practical Guide

| Scenario | Which Schema? | Why? |
|----------|---------------|------|
| **Real-time dashboard for the farm manager watching oxygen levels and feed rates on their iPad** |  **STAR** | They need instant responses to save fish from hypoxia. **Speed is life.** |
| **Scientific research over 10 years, tracking genetic lineages, complex veterinary treatments, and shifting ocean currents** |  **SNOWFLAKE** | We have deeply nested hierarchies (e.g., Treatment → Vet_Visit → Diagnosis → Pathogen_Strain). We need strict data cleanliness, and we can wait 5 seconds for a complex report. |
| **Daily operational reports (e.g., "Feed conversion ratio per cage for today")** |  **STAR** | Our farm workers don't know SQL; they just drag and drop in Power BI. Flat tables are far easier for them to understand. |

---

### The Modern Aquaculture Compromise

Most companies use a **Star Schema** for their daily operational dashboards, but keep a separate **Snowflake "Data Vault"** in the background specifically for tracking the complex regulatory compliance data (like veterinary drug withdrawal periods), where every historical change must be meticulously preserved without duplication.

