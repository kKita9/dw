## 1) Goal

The goal of this model is to analyze **daily box office performance** and slice results by **time**, **movie attributes** (from OMDb), and **distributor**.

Typical questions:

* daily / monthly / yearly revenue trends
* top movies by revenue
* revenue by distributor
* revenue by genre

## 2) Data sources and mapping

* **revenues_per_day.csv** → `FACT_BOX_OFFICE_DAILY`

  * provides daily metrics: revenue and theaters
  * provides business context: movie title and distributor name (used to build dimensions)
* **OMDb API** → `DIM_MOVIE` (+ `DIM_GENRE`)

  * provides descriptive attributes for a movie (year, runtime, director, etc.)
  * provides stable identifier: `imdb_id`

## 3) Fact table

### `FACT_BOX_OFFICE_DAILY`

**Grain (row level)**

* 1 row = **movie + day + distributor**

**Keys (FK)**

* `date_key` → `DIM_DATE`
* `movie_key` → `DIM_MOVIE`
* `distributor_key` → `DIM_DISTRIBUTOR`

**Measures**

* `revenue_amount`
* `theaters_cnt`

**Data quality rule**

* enforce uniqueness for the grain:

  * `UNIQUE(date_key, movie_key, distributor_key)`

## 4) Dimensions

### `DIM_DATE`

One row per calendar day.   

* `date_key` (PK)
* `date`, `year`, `month`, `day`, `week_of_year`, `day_of_week`, `quarter`, `is_weekend`

### `DIM_DISTRIBUTOR`

Dictionary of distributors.

* `distributor_key` (PK)
* `distributor_name`

### `DIM_MOVIE`

One row per movie (identified by `imdb_id`).

* `movie_key` (PK)
* `imdb_id` (business key, should be unique)
* main attributes: `title`, `year`, `type`, `released_date`, `runtime_min`
* slicing attributes: `country`, `language`, `director`, `writer`, `actors`
* ratings: `imdb_rating`, `imdb_votes`, `metascore`
* extra: `awards`

## 5) Genre modeling (many-to-many)

A movie can have **multiple genres**, and each genre can be linked to **many movies**.

### `DIM_GENRE`

* `genre_key` (PK)
* `genre_name`

### `BRIDGE_MOVIE_GENRE`

* `movie_key` (FK → `DIM_MOVIE`)
* `genre_key` (FK → `DIM_GENRE`)
* `PK(movie_key, genre_key)` to prevent duplicates

This design enables clean filtering and grouping by genre without parsing comma-separated strings.

## 6) Assumptions and limitations

* `revenue_amount` currency is not explicitly provided in the sources, so the model treats it as a **single consistent currency**.
* Only genre is normalized via bridge; other multi-value attributes (like actors) are kept as strings for simplicity (can be normalized later if needed).
