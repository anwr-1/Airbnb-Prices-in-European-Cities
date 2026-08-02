# 🏠 Airbnb Prices in European Cities

An end-to-end data pipeline analyzing ~51,700 Airbnb listings across 10
European cities — from raw data through cleaning, enrichment, a SQL Server
star schema, and a Power BI dashboard.

**Cities:** Amsterdam, Athens, Barcelona, Berlin, Budapest, Lisbon, London, Paris, Rome, Vienna

## Pipeline

```
Old Data/ (raw)  →  cleaning.ipynb  →  airbnb_europe_clean.csv  →  SQL star schema  →  Power BI
                                    ↘  airbnb_scraper.py (enrichment) → airbnb_scraped_reviews.csv
```

1. **Raw data** (`Old Data/`) — 20 CSVs (10 cities × weekday/weekend) from the
   [Airbnb Prices in European Cities](https://www.kaggle.com/datasets/thedevastator/airbnb-prices-in-european-cities)
   Kaggle dataset (originally Gyódi & Nawaro, 2021).
2. **Cleaning** (`Modified Data/cleaning.ipynb`) — merges all 20 files,
   drops redundant columns, fixes types, and engineers new features:
   - `price_per_night` / `price_per_person`
   - `host_type` (Professional / semi_professional / individual, from the
     `biz`/`multi` flags)
   - `location_category` (city_center / near_center / suburban / outskirts,
     binned from distance to city center)
   - `is_price_outlier` (top 1% of prices flagged)

   Output: `airbnb_europe_clean.csv` (51,708 rows, 19 columns).
3. **Enrichment** (`Modified Data/airbnb_scraper.py`) — a Selenium scraper
   that visits live Airbnb listing pages for a sample of properties and
   pulls review count + rating, producing `airbnb_scraped_reviews.csv`
   (496 enriched listings).
4. **Data warehouse** (`SQL/`) — a star schema in SQL Server:
   `fact_listings` plus `dim_city`, `dim_property_type`, `dim_room_type`,
   `dim_host`, `dim_date`. `create_star_schema.sql` builds it,
   `populate_star_schema.sql` loads it from the staging table.
5. **Dashboard** (`DashBoard/AirbnbDashBoard.pbix`) — Power BI report built
   on top of the star schema.

## Project structure

```
Airbnb Prices in European Cities/
├── Old Data/                       # raw per-city CSVs (bronze)
├── Modified Data/
│   ├── cleaning.ipynb              # raw → clean ETL
│   ├── airbnb_europe_clean.csv     # cleaned dataset
│   ├── airbnb_scraper.py           # Selenium review/rating scraper
│   └── airbnb_scraped_reviews.csv  # enriched sample
├── SQL/
│   ├── create_star_schema.sql
│   └── populate_star_schema.sql
└── DashBoard/
    └── AirbnbDashBoard.pbix
```

## Reproducing it

1. Run `cleaning.ipynb` (update `bronze_layer_path` to point at `Old Data/`
   on your machine — it's currently a hardcoded local path) to regenerate
   `airbnb_europe_clean.csv`.
2. *(Optional)* Run `airbnb_scraper.py` to refresh the review/rating sample.
   Requires `selenium` + `webdriver-manager` and a local Chrome install.
3. Run `create_star_schema.sql` in SQL Server, import `airbnb_europe_clean.csv`
   into `staging_listings`, then run `populate_star_schema.sql`.
4. Open `AirbnbDashBoard.pbix` in Power BI Desktop, pointed at the database.

## Tech stack

Python (pandas, Selenium) · SQL Server (T-SQL) · Power BI

## License

MIT
