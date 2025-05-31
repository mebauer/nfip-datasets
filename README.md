# How to Download NFIP Datasets from OpenFEMA
Author: Mark Bauer

# Overview
This repository demonstrates how to download and work with the National Flood Insurance Program (NFIP) [datasets](https://www.fema.gov/about/openfema/data-sets#nfip) from [OpenFEMA](https://www.fema.gov/about/reports-and-data/openfema). The workflows use the Python programming language and are sdesigned to be efficient, easy to adapt, and easy to understand. These examples also include brief data exploration using [DuckDB](https://duckdb.org/).

The tutorial can be found in the [download-examples](https://github.com/mebauer/nfip-datasets/blob/main/download-examples.ipynb) notebook.

Notes:  
- Datasets are downloaded in Parquet format whenever possible for performance and compatibility.
- DuckDB is used for querying and previewing the data.
- The NFIP Community Layers dataset is large and has been excluded from this repository (tracked via .gitignore), but other datasets are included both locally and on GitHub.
- For the NFIP Policies and Claims datasets, the current focus is on a sample of New York City.

# Disclaimer
This tutorial uses the Federal Emergency Management Agency’s OpenFEMA API, but is not endorsed by FEMA. The Federal Government or FEMA cannot vouch for the data or analyses derived from these data after the data have been retrieved from the Agency's website(s).

Read more about OpenFEMA's [Terms and Conditions](https://www.fema.gov/about/openfema/terms-conditions).

# Datasets
- [FIMA NFIP Redacted Claims - v2](https://www.fema.gov/openfema-data-page/fima-nfip-redacted-claims-v2)
- [FIMA NFIP Redacted Policies - v2](https://www.fema.gov/openfema-data-page/fima-nfip-redacted-policies-v2)
- [NFIP Multiple Loss Properties - v1](https://www.fema.gov/openfema-data-page/nfip-multiple-loss-properties-v1)
- [NFIP Residential Penetration Rates - v1](https://www.fema.gov/openfema-data-page/nfip-residential-penetration-rates-v1)
- [2023 NFIP Reinsurance Placement Information](https://www.fema.gov/about/openfema/data-sets/national-flood-insurance-program-nfip-reinsurance-placement-information)
- [NFIP Community Layer Comprehensive - v1](https://www.fema.gov/openfema-data-page/nfip-community-layer-comprehensive-v1)
- [NFIP Community Layer No Overlaps Split - v1](https://www.fema.gov/openfema-data-page/nfip-community-layer-no-overlaps-split-v1)
- [NFIP Community Layer No Overlaps Whole - v1](https://www.fema.gov/openfema-data-page/nfip-community-layer-no-overlaps-whole-v1)
- [NFIP Community Status Book - v1](https://www.fema.gov/openfema-data-page/nfip-community-status-book-v1)

# Additional Resources
- [OpenFEMA](https://www.fema.gov/about/reports-and-data/openfema)
- [OpenFEMA Datasets](https://www.fema.gov/about/openfema/data-sets)
- [OpenFEMA National Flood Insurance Program Datasets](https://www.fema.gov/about/openfema/data-sets#nfip)
- [OpenFEMA API Documentation](https://www.fema.gov/about/openfema/api)
- [OpenFEMA Developer Resources](https://www.fema.gov/about/openfema/developer-resources)
- [OpenFEMA Guide to Working with Large Data Sets](https://www.fema.gov/about/openfema/working-with-large-data-sets)

# Say Hello!
Feel free to reach out.
- LinkedIn: [markebauer](https://www.linkedin.com/in/markebauer/)   
- Portfolio: [mebauer.github.io](https://mebauer.github.io/)
- GitHub: [mebauer](https://github.com/mebauer) 
