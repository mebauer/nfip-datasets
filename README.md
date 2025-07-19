# How to Download NFIP Datasets from OpenFEMA
Author: Mark Bauer

# Overview
This repository demonstrates how to download and work with the National Flood Insurance Program (NFIP) [datasets](https://www.fema.gov/about/openfema/data-sets#nfip) from [OpenFEMA](https://www.fema.gov/about/reports-and-data/openfema). The workflows use the Python programming language and are designed to be efficient, easy to adapt, and easy to understand. These examples also include brief data exploration using [DuckDB](https://duckdb.org/).

The tutorial can be found in the [download-examples](https://github.com/mebauer/nfip-datasets/blob/main/download-examples.ipynb) notebook.

Notes:  
- Datasets are downloaded in Parquet format whenever possible for performance and compatibility.
- DuckDB is used for querying and previewing the data.
- The NFIP Community Layers datasets are large and have been excluded from this repository (tracked via .gitignore), but other datasets are included both locally and on GitHub.
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


<details>
<summary>📄 Click to view full source code: <code>fetch_nfip_data()</code></summary>

<div style="max-height: 500px; overflow: auto; border: 1px solid #ddd; padding: 10px; margin-top: 10px;">

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
                    for chunk in response.iter_content(chunk_size=8192):
                        f.write(chunk)
            except Exception as e:
                logging.error(f"Request failed at skip={skip}: {e}")
                raise

            try:
                con = duckdb.connect()
                result = con.execute(f"SELECT COUNT(*) FROM read_parquet('{page_path}')").fetchone()
                page_count = result[0]
                con.close()
            except Exception as e:
                logging.error(f"Parquet read failed at skip={skip}: {e}")
                os.remove(page_path)
                raise

            if page_count == 0:
                logging.info(f"No more data returned at skip={skip}. Ending.")
                os.remove(page_path)
                break

            total_records += page_count
            logging.info(f"Page loaded: {page_count} records")

            if page_count < PAGE_SIZE:
                logging.info("Final page detected.")
                break

            skip += PAGE_SIZE
            time.sleep(sleep_secs)

        if total_records > 0:
            try:
                con = duckdb.connect()
                input_glob = os.path.join(tmpdir, "*.parquet")
                con.execute(f"""
                    COPY (SELECT * FROM read_parquet('{input_glob}'))
                    TO '{output_file}' (FORMAT PARQUET)
                """)
                logging.info(f"Wrote {total_records:,} records to {output_file}")
                con.close()
            except Exception as e:
                logging.error(f"Failed to write output Parquet: {e}")
                raise
        else:
            logging.warning("No data fetched. Nothing saved.")
</div> </details>```

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
