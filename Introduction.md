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
