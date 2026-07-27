# Data Warehouse Architectures

How we physically build and populate our warehouse defines its architecture. There are three classic approaches, plus the modern twist:

---

## A. Inmon's Top-Down

**Concept:** Build the massive, normalized, enterprise-wide warehouse first (the "Single Source of Truth"). Then, build smaller departmental data marts from this central warehouse. Think of it as building the main engine of a factory first, then attaching specific conveyor belts (marts) to it.

**Aquaculture Example:** We build a giant central warehouse with 500 tightly normalized tables (e.g., Farm, Pen, Fish_Batch, Feed_Order, Health_Event). Only after this is complete, we slice out a "Logistics Mart" for the harvest team. This is great for data consistency but takes 18 months to build.

**Pros:** Highly consistent, minimal data redundancy.

**Cons:** Very slow and expensive to build; rigid to change.

---

