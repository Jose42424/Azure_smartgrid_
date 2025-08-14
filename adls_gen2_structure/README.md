# Azure Data Lake Storage Gen2 Structure

This folder contains a sample representation of the ADLS Gen2 zone-based architecture used in the Smart Grid project.

## Zones
- **raw/**: Direct event ingestion from Event Hubs (unvalidated, unprocessed data).
- **clean/**: Data after basic validation and cleaning.
- **curated/**: Final, analytics-ready datasets.

## Notes
- Only small samples are stored here for illustration.
- Full datasets are stored securely in the ADLS Gen2 container:
  - Storage account: `smartgridstoragejm`
  - Container: `smartgrid`
  - Path: `smartgridstpragejm/smartgrid/...`
