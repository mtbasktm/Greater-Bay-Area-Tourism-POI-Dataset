Greater Bay Area Tourism POI Dataset
[English](README.md) | [简体中文](README.zh-CN.md)

A tourism-oriented Point of Interest (POI) dataset covering the 11 cities of the Guangdong–Hong Kong–Macao Greater Bay Area (GBA). It contains attraction names, addresses, geographic coordinates, categories, provenance metadata, and source-specific attributes in JSON format.

Important: This is a collected sample, not an exhaustive registry of GBA attractions. The dataset contains known geographic misclassifications and third-party source material. Review Data quality and Source and redistribution before using or publishing it.
Dataset overview
Item	Value
Data file	[`data/pois.csv`](data/pois.csv)
Records	3,597
Unique poi_key values	3,597
Cities	11
CSV columns	12
Nonempty category labels	35
Missing/empty category	1,101 (30.61%)
first_seen_at range	2026-09-30 to 2026-10-16 (UTC+08:00)
File size	4,335,888 bytes (~4.34 MB)
CSV encoding	UTF-8 with BOM (utf-8-sig)


first_seen_at records when a POI was first stored in the acquisition database; it is not necessarily the date when the POI was created, visited, or last updated by the source.
Coverage by city
City	POIs
Guangzhou (广州)	859
Shenzhen (深圳)	445
Zhongshan (中山)	368
Huizhou (惠州)	342
Dongguan (东莞)	304
Jiangmen (江门)	291
Foshan (佛山)	259
Zhuhai (珠海)	254
Zhaoqing (肇庆)	219
Hong Kong (香港)	151
Macao (澳门)	105
Total	3,597


These figures are record counts in this file, not estimates of each city's actual number of attractions. In particular, some records associated with Zhongshan are demonstrably outside the GBA (see below).
Data dictionary
Column	Description	Example / notes
poi_key	Dataset-level primary key, constructed using the source prefix and platform ID	flyai:67058
platform_id	POI identifier returned by the source platform	67058
city	City assigned to the record during acquisition/import; may be incorrect	广州
name	Name of the attraction or POI	流溪河国家森林公园
address	Source-provided POI address (trimmed during normalization)	Chinese address string
latitude	Latitude as a decimal number	23.74822
longitude	Longitude as a decimal number	113.782528
category	Source-provided attraction category; may be empty	森林丛林
location_status	Historical city-validation label from the collection process	city_confirmed, mismatch, unknown
first_source	First ingestion route or import origin	search-poi, v7_baseline
first_seen_at	Timestamp when the record was first inserted into the dataset	ISO 8601 with timezone
raw_json	JSON string preserving the original per-POI attributes	Parsed JSON object


The coordinate reference system (e.g., WGS84 versus GCJ-02) has not been independently verified. Do not perform precision-critical geospatial analysis until it is confirmed.
Attributes inside raw_json
The per-record JSON may include id, name, address, latitude, longitude, category, description, mainPic (image URL), jumpUrl (source detail link), freePoiStatus, poiLevel, listRank, and ticketInfo. Some baseline imports also preserve district, source, sourceTool, sourceEndpoint, occurrenceCount, matchedQueries, and retrievedAt.

These fields are optional or nullable, and their presence varies by source and record. Ticket information, attraction status, rankings, and URLs may become outdated. Images are referenced by URL and are not included as files in this dataset.
Example usage
The dataset can support exploratory tourist-attraction discovery, retrieval-augmented generation (RAG), candidate generation for travel agents, text-based recommendation experiments, geographic visualization, and research on POI quality and entity resolution.
import json
import pandas as pd

pois = pd.read_csv("data/pois.csv", encoding="utf-8-sig")

# Example: Guangzhou museums with a confirmed city-validation label.
subset = pois.loc[
    (pois["city"] == "广州")
    & (pois["category"] == "博物馆")
    & (pois["location_status"] == "city_confirmed"),
    ["poi_key", "name", "address", "latitude", "longitude"],
]
print(subset.head())

# Extract the optional source description from a JSON field.
pois["description"] = pois["raw_json"].apply(
    lambda value: json.loads(value).get("description")
)
Note: The city_confirmed filter is not sufficient to guarantee geographic correctness. Independent spatial validation is required for location-sensitive applications.
Data quality and known issues
The exported file contains the following historical validation labels:
location_status	Records	Share
city_confirmed	3,541	98.44%
mismatch	44	1.22%
unknown	12	0.33%


Known limitations:
1. False positive city validation: at least 34 records assigned to Zhongshan (中山) have coordinates far outside the GBA (around Dalian, Liaoning), yet are labeled city_confirmed. For example, 大连港 is assigned to 中山 despite having an address in Dalian's Zhongshan District. Another out-of-region record is labeled mismatch. Similar city/district names can cause incorrect search matches; the precise cause inside the validation module has not been confirmed.
2. Missing categories: 1,101 records have an empty category value. An empty category is not equivalent to an unknown POI type verified by human annotation.
3. Incomplete geographic accuracy assessment: city_confirmed is a historical rule-based label, not a guarantee of correct coordinates, city, or entity identity.
4. Deduplication scope: records are unique by poi_key / platform ID. Different IDs may still refer to the same real-world place; name-level and spatial entity resolution have not been completed.
5. Sampling bias: counts reflect the acquisition strategy and API results. They should not be used to infer true city-level attraction density or evaluate population-level coverage without a defensible sampling framework.
6. Potentially stale attributes: source descriptions, URLs, prices, rankings, and status values are historical snapshots, not live data.
For spatial research, we recommend an additional city polygon/bounding-box check, administrative-address parsing, and human review of suspicious records. Do not silently rewrite the original source fields when cleaning the dataset; retain a provenance trail.
Data origin and reproducibility
The records originate from a FlyAI / Fliggy-related POI acquisition workflow. In this exported file:
first_source	Records	Interpretation
search-poi	2,914	Ingested through the structured POI search route
v7_baseline	683	Imported from an earlier baseline dataset


The associated crawler.py implements a multi-query acquisition pipeline around the FlyAI CLI: query planning, search-poi calls, response parsing (data.itemList), POI normalization, SQLite storage, deduplication by platform ID, quality-assessment integration, and CSV export. The single script is not a standalone, fully reproducible crawler: its configuration, supporting Python modules, query input, authorized API access, and the historical baseline input are required. Re-running it later may yield different POIs because upstream responses and access limits can change.

Official references:
- Fliggy/FlyAI developer documentation
- Official FlyAI search-poi command documentation
- Official FlyAI CLI project
Source and redistribution
The dataset includes source-provided attraction descriptions, links, and metadata; some records include commercial/third-party image and ticket URLs. The upstream FlyAI code being open source does not grant a redistribution license to the underlying POI records or descriptions. Authorization to distribute this CSV publicly, or to reuse its text/images in training or commercial systems, has not been established by this README.

Before uploading the CSV to a public repository, check the applicable API/data terms and obtain any necessary permission. If these rights cannot be confirmed, publish only an authorized or appropriately derived subset and keep the full source export private. No open-data license for pois.csv is asserted here.
Citation
If used in research, identify the dataset as a historical GBA tourism-POI collection derived from the FlyAI-related acquisition workflow, indicate the version/date accessed, and document any corrections or exclusions made during preprocessing. A formal citation identifier or DOI has not been assigned.
