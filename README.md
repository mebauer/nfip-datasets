# 📥 Downloading National Flood Insurance Program (NFIP) Datasets with OpenFEMA API
Author: Mark Bauer

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

# 📌 Overview
This repository demonstrates how to download and work with the National Flood Insurance Program (NFIP) [datasets](https://www.fema.gov/about/openfema/data-sets#nfip) from [OpenFEMA](https://www.fema.gov/about/reports-and-data/openfema). The workflows use the Python programming language and are designed to be efficient, easy to adapt, and easy to understand. These examples also include brief data exploration using [DuckDB](https://duckdb.org/).

📥 The tutorial can be found in the [download-examples](https://github.com/mebauer/nfip-datasets/blob/main/download-examples.ipynb) notebook.

Notes:  
- 📦 Datasets are downloaded in Parquet format whenever possible for performance and compatibility.
- 🐤 DuckDB is used for querying and previewing the data.
- 🚫 The NFIP Community Layers datasets are large and have been excluded from this repository (tracked via .gitignore), but other datasets are included both locally and on GitHub.
- 🗽 For the NFIP Policies and Claims datasets, the current focus is on a sample of New York City.

# ⚠️ Disclaimer
This tutorial uses the Federal Emergency Management Agency’s OpenFEMA API, but is not endorsed by FEMA. The Federal Government or FEMA cannot vouch for the data or analyses derived from these data after the data have been retrieved from the Agency's website(s).

⚖️ Read more about OpenFEMA's [Terms and Conditions](https://www.fema.gov/about/openfema/terms-conditions).

# 🌊 OpenFEMA API and the NFIP Claims and Policies Datasets
The NFIP Claims and Policies datasets are among the most widely used resources provided by OpenFEMA. However, these files are often very large, and bulk downloads can require significant storage and processing time.

🎯 If you're only interested in a few counties, it's more efficient to use the OpenFEMA API to retrieve just the data you need. The snippet below shows a simplified version of a function that fetches paginated NFIP data for a given county using the OpenFEMA API.

<details>
<summary>📄 Expand to see a preview of <code>fetch_nfip_data()</code></summary>

```python
# constants
BASE_URL = "https://www.fema.gov/api/open/v2/FimaNfip{}"
PAGE_SIZE = 10_000
ALLOWED_DATASETS = {"Claims", "Policies"}

# configure logging
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s"
)

# build OpenFEMA API URL
def build_url(dataset, fips, skip):
    return (
        f"{BASE_URL.format(dataset)}"
        "?$format=parquet"
        f"&$filter=countyCode eq '{fips}'"
        f"&$top={PAGE_SIZE}"
        f"&$skip={skip}"
    )

# download and save NFIP data
def fetch_nfip_data(dataset, fips, outpath=".", sleep_secs=2.0):
    """
    Downloads FEMA NFIP data (Claims or Policies) for a given county FIPS code,
    handling pagination and writing the result as a single Parquet file.
    """
    dataset_cap = dataset.capitalize()
    if dataset_cap not in ALLOWED_DATASETS:
        raise ValueError(f"Invalid dataset '{dataset}'. Must be one of {ALLOWED_DATASETS}")

    os.makedirs(outpath, exist_ok=True)
    output_file = os.path.join(outpath, f"{dataset.lower()}-{fips}.parquet")

    with tempfile.TemporaryDirectory() as tmpdir:
        skip = 0
        total_records = 0
        logging.info(f"Starting download for FIPS {fips}, dataset '{dataset_cap}'")

        while True:
            url = build_url(dataset_cap, fips, skip)
            logging.info(f"Requesting page: skip={skip}")
            page_path = os.path.join(tmpdir, f"page_{skip}.parquet")

            try:
                response = requests.get(url, timeout=60)
                response.raise_for_status()
                with open(page_path, "wb") as f:
    
    ...   
```
</details>

To download data, simply call the `fetch_nfip_data()` function and pass in your desired county FIPS codes.

The example below downloads NFIP **Claims** data for the five counties that make up 🗽 New York City:

```python
# specify County FIPS codes
fips_list = [
    "36005", # Bronx
    "36047", # Kings (Brooklyn)
    "36061", # New York (Manhattan)
    "36081", # Queens
    "36085" # Richmond (Staten Island)
]

# Claims or Policies
dataset = "Claims"

# specify outpath
outpath = "data/"

# loop over County FIPS codes
for fips in fips_list:
    
    # pass variables to function
    fetch_nfip_data(dataset, fips, outpath)
```

🔧 Refer to the full function inside the [download-examples](https://github.com/mebauer/nfip-datasets/blob/main/download-examples.ipynb) notebook. Note: We download data as Parquet files for performance. With DuckDB, these can easily be converted to CSV. Please refer to the [parquet-to-csv](https://github.com/mebauer/nfip-datasets/blob/main/parquet-to-csv.ipynb) notebook.

🗺️ If you're interested in analyzing data at the national scale, check out my project, [Analyzing FEMA's National Flood Insurance Program (NFIP) Data With DuckDB](https://github.com/mebauer/duckdb-fema-nfip). That example demonstrates how to download the full dataset as a Parquet file and efficiently query it using DuckDB.

# 📂 Available Datasets
- [FIMA NFIP Redacted Claims - v2](https://www.fema.gov/openfema-data-page/fima-nfip-redacted-claims-v2)
- [FIMA NFIP Redacted Policies - v2](https://www.fema.gov/openfema-data-page/fima-nfip-redacted-policies-v2)
- [NFIP Multiple Loss Properties - v1](https://www.fema.gov/openfema-data-page/nfip-multiple-loss-properties-v1)
- [NFIP Residential Penetration Rates - v1](https://www.fema.gov/openfema-data-page/nfip-residential-penetration-rates-v1)
- [2023 NFIP Reinsurance Placement Information](https://www.fema.gov/about/openfema/data-sets/national-flood-insurance-program-nfip-reinsurance-placement-information)
- [NFIP Community Layer Comprehensive - v1](https://www.fema.gov/openfema-data-page/nfip-community-layer-comprehensive-v1)
- [NFIP Community Layer No Overlaps Split - v1](https://www.fema.gov/openfema-data-page/nfip-community-layer-no-overlaps-split-v1)
- [NFIP Community Layer No Overlaps Whole - v1](https://www.fema.gov/openfema-data-page/nfip-community-layer-no-overlaps-whole-v1)
- [NFIP Community Status Book - v1](https://www.fema.gov/openfema-data-page/nfip-community-status-book-v1)
   
# 📚 Additional Resources
- [OpenFEMA](https://www.fema.gov/about/reports-and-data/openfema)
- [OpenFEMA Datasets](https://www.fema.gov/about/openfema/data-sets)
- [OpenFEMA National Flood Insurance Program Datasets](https://www.fema.gov/about/openfema/data-sets#nfip)
- [OpenFEMA API Documentation](https://www.fema.gov/about/openfema/api)
- [OpenFEMA Developer Resources](https://www.fema.gov/about/openfema/developer-resources)
- [OpenFEMA Guide to Working with Large Data Sets](https://www.fema.gov/about/openfema/working-with-large-data-sets)

# 👋 Stay in Touch
Feel free to reach out.
- LinkedIn: [markebauer](https://www.linkedin.com/in/markebauer/)   
- Portfolio: [mebauer.github.io](https://mebauer.github.io/)
- GitHub: [mebauer](https://github.com/mebauer) 
