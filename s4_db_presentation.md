% SQL and Databases for Research
% Eric Seymour, PhD
% September 13, 2017


## Workshop Overview

* Overview of databases (DBs)
* Overview of SQL syntax
* Overview of DB creation
* Spatial databases and spatial queries
* Interacting w/ (spatial) DBs using QGIS and Python 

---

## Database Overview

> - DBs good for storing and working with large datasets
> - DBs keep related data files in a single place
> - DBs let you link tables using common fields or spatial relationships
> - DBs enforce data integrity, e.g., only numbers in number column
> - SQL can be stored in text files (research reproducibility)

---

## Database Design

* DBs contain one or more tables
* Tables consist of columns (fields) and rows (observations)
* Columns have explicitly defined column types (text, int, real)
* Tables can be populated from text files (usually CSV files)

---

## Database Software

| Name           | Type         |
|:---------------|:-------------|
| MS Access      | File based   |
| SQLite         | File based   |
| MS SQL Server  | Server based |
| MySQL (Oracle) | Server based |
| PostgreSQL     | Server based |


---

## SQLite

* Does not require a server process
* No configuration required 
* Self-contained, no external dependencies
* Works on all platforms

---

## SQLite Usage

* Command line interface (CLI) [sqlite3]((https://sqlite.org/index.html))
* [Firefox SQLite Manager plugin](https://addons.mozilla.org/en-US/firefox/addon/sqlite-manager/)
* Plugin needs to be installed for workshop

---

## Command Line Shell Program

* Linux: `apt-get install sqlite3`
* OSX: `brew install sqlite3`
* Windows: download binaries; add sqlite to PATH
	- see [here](https://www.codeproject.com/Articles/850834/Installing-and-Using-SQLite-on-Windows) for details

---

## Creating Tables

* SQLite Manager simplifies table creation with wizard
* Table creation requires a CREATE TABLE statement using SQL
* These statements define the table name, field names, and field types
* Also identifies keys and constraints (optional)

---

## Simple Table Creation

* Open SQLite Manager
* Select Databse -> New In-Memory Database
* Enter code in dialog box in 'Execute SQL' tab, hit RUN SQL

```sql
CREATE TABLE cars (
id      INTEGER PRIMARY KEY, 
make    TEXT,
model   TEXT,
price   INTEGER);

```

---

## Add data

* Remove CREATE code
* Copy statement below into dialog box, hit RUN SQL

```sql
INSERT INTO cars VALUES (1, 'Acura', 'NSX', 47045);
INSERT INTO cars VALUES (2, 'Audi', 'A8', 63890);
INSERT INTO cars VALUES (3, 'BMW', 'X1', 108900);

```

---

## View Table

This query returns all columns and rows

```sql
SELECT *    /* '*' means ALL columns */
FROM cars;  
```


---

### SQL: Structured Query Language

```sql
SELECT field1, field2   /* specify columns */
FROM myTable;           /* specify tables   */
WHERE condition = 'met'; /* filter (optional) */
```

* SELECT and FROM are mandatory elements
* Many optional clauses, including WHERE
* Individual queries end with `;`

---

## Operators in WHERE Clause

| Operator | Description                                     |
|:---------|:------------------------------------------------|
| =        | Equal                                           |
| <>       | Not equal                                       |
| >(=)     | Greater than (or equal)                         |
| <(=)     | Less than (or equal)                            |


---

## WHERE Clause, Cont.

| Operator | Description                                     |
|:---------|:------------------------------------------------|
| BETWEEN  | Between an inclusive range                      |
| LIKE     | Search for a pattern                            |
| IN       | To specify multiple possible values for a colum |

---

## SQL Challenge

Find make, model, price for cars that cost 50,000 or more

. . .

```sql
SELECT make, model, price
FROM cars
WHERE price >= 50000;

```

---

## Order Results

* Use ORDER clause to sort results by field(s)
* Default is ASC values; can also specify DESC
* Let's ORDER results of entire table by price

```sql
SELECT *
FROM cars
ORDER BY price DESC;
```

---

## SQL Functions

* SQL can returns summary information
* MIN, MAX, COUNT, AVG, SUM
* You need to wrap the field in the function

```sql
SELECT COUNT(*), AVG(price), MIN(price), MAX(price)
FROM cars;
```

---

## SQL Functions, cont.

* Many string/text manipulation functions
* Let's find all cars whose manufacturer starts with 'A'
* Add SQL SUBSTR function to query
* Takes 2 arguments: position, length

```sql
SELECT *
FROM cars
WHERE SUBSTR(make,1,1) = 'A'
ORDER BY make;
```

---

## LIKE Function

* We could use LIKE for same purpose
* '%' is the wildcard character 

```sql
SELECT *
FROM cars
WHERE make LIKE 'A%'
ORDER BY make;
```

---

## SQL Core Functions

* Many functions for operating on text and numbers
* Useful for cleaning data and customizing output
* [Full list for SQLite core functions](https://sqlite.org/lang_corefunc.html)


---

## Deleting Tables

Not Reversible

```sql
DROP TABLE a_cars;
```

---

## Tip of the SQL/ Database Iceberg

* You now know how to create and query a database table
* Everything else builds on these fundamental ideas
* Are there questions before we move on?

---

# Table Join

---

## Adding a second table

Create new table of safety ratings for cars

```sql
CREATE TABLE ratings (
id      INTEGER PRIMARY KEY,
make    TEXT,
model   TEXT,
rating  TEXT);
```

---

## Add data

Cars have hypothetical A to F ratings

```sql
INSERT INTO ratings VALUES (1, 'Acura', 'NSX', 'C');
INSERT INTO ratings VALUES (2, 'Audi', 'A8', 'A');
INSERT INTO ratings VALUES (3, 'BMW', 'X1', 'B');
```

---

## Basic Table Joins

* Join tables on common field
* We have several fields in common in this case
* Join on id field in both tables

```sql
SELECT A.make, A.model, B.rating
FROM cars AS A JOIN ratings AS B /* AS alias */
ON A.id = B.id; /* specify field to join on */
```

---

## Types of Joins in SQLite

* Inner Join (Default)

Returns all rows from multiple tables where the join condition is met

. . .

* LEFT OUTER JOIN

Returns __all__ rows from the LEFT-hand table specified in the ON condition and only those rows from the other table where the joined fields are equal

---

![Inner Join](figures/inner_join.png)

---

![Left Outer Join](figures/outer_join.png)

---


## Loading CSV Files

* Read CSV files in using command line interface
* Program defaults data to TEXT

```bash
sqlite> .mode csv
sqlite> .import ACS_15_5YR_B25003.csv wayne_tenure
sqlite> SELECT COUNT(*) FROM wayne_tenure;
611
```

---

## Read CSV files using Python

```python
import sqlite3 as sql
import csv

con = sql.connect(":memory:")
cur = con.cursor()
cur.execute('''
	CREATE TABLE wayne_tenure (
	geoid TEXT,
	geoid2 TEXT,
	geoid_label TEXT,
	HD01_VD01 INT 
	HD02_VD01 INT
	HD01_VD02 INT
	HD02_VD02 INT
	HD01_VD03 INT
	HD02_VD03 INT
	);
	''')

with open("ACS_15_5YR_B25003.csv", "r") as f:
	reader = csv.reader()
	header = reader.next()
	for row in reader:
		cur.execute('''
			INSERT INTO wayne_tenure
			VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?);
			''', row)

con.commit()
con.close()

```

---

# Spatial Databases

---

## Spatial Databases

* We can use coordinate/geometry data to make spatial tables
* SQLite -> Spatialite
* Special spatial SQL syntax for querying for relationships based on location
* QGIS makes it easy to create and work w/ Spatialite DBs

---

## Tips

* Begin project with spatial table to initialize spatial components of DB
* Then add spatial and non-spatial tables in any order
* Use CLI or GUI to generate DB from initial spatial layer
* Export shp to new spatialite DB via QGIS export

---

## Spatialite Tools

* Command line tool for reading shapefiles into spatial DB table
* NB: This tool is inferior to others

```bash
spatialite_tool -i -shp tl_2010_26163_tract10 \\
-d test.sqlite -t wayne_tracts \\
-c CP1252 -s 2898
```

---

## ogr2ogr

CLI installed with GDAL: `apt-get install gdal-bin`


```bash
ogr2ogr -f 'SQLite' -dsco SPATIALITE=YES -t_srs EPSG:2898 \\
test.sqlite tl_2010_26_place10.shp -nln mi_places -nlt PROMOTE_TO_MULTI
```

```bash
ogr2ogr -f 'SQLite' -update -t_srs EPSG:2898 \\
test.sqlite tl_2010_26163_tract10.shp -nln wayne_tracts -nlt PROMOTE_TO_MULTI
```

---

## Tenure Heat Map 

>- Run python script to add tenure data to test.sqlite
>- Open QGIS DB Manager
>- JOIN tenure data to tract geos
>- Create choropleth map

---

* Join on fields containing full tract FIPS
* Select calculated field AND geometry

```sql
SELECT A.geoid2, 
A.HD01_VD02 * 1.0 / A.HD01_VD01 * 100 AS own_rate,
B.geometry
FROM wayne_tenure AS A JOIN wayne_tracts AS B
ON A.geoid2 = B.GEOID10;
```
---

## Spatial SQL Functions

* Spatialite implements a subset of PostGIS functions
* Functions correspond to many vector tools in ArcGIS
* Many begin with "ST" for "space-time"
* [Full list](https://www.gaia-gis.it/gaia-sins/spatialite-sql-4.3.0.html#p1)

---

## Find Tract Centroids

```sql
SELECT A.geoid2, 
A.HD01_VD02 * 1.0 / A.HD01_VD01 * 100 AS own_rate,
st_centroid(B.geometry) /* wrap geom in centroid function */
FROM wayne_tenure AS A JOIN wayne_tracts AS B
ON A.geoid2 = B.GEOID10;
```

---

## Buffer Centroids

```sql
SELECT A.geoid2, 
A.HD01_VD02 * 1.0 / A.HD01_VD01 * 100 AS own_rate,
st_buffer(st_centroid(B.geometry), 1000) /* dist in CRS units (ft) */
FROM wayne_tenure AS A JOIN wayne_tracts AS B
ON A.geoid2 = B.GEOID10;
```

---

## Spatial Queries

* Find all tracts in Detroit
* Add census places layer

```bash
spatialite_tool -i -shp tl_2010_26_place10 \\
-d test.sqlite -t mi_tracts \\
-c CP1252 -s 2898
```
---

## Spatial Queries

```sql
select A.*
from wayne_tracts AS A, mi_places AS B
WHERE st_contains(B.geometry, A.geometry)
AND B.placefp10 = '22000';
```

---

## Spatial Index

* Speed performance
* Require explicit inclusion in query

```sql
select A.*
from wayne_tracts AS A, mi_places AS B
WHERE st_contains(B.geometry, A.geometry)
AND A.ROWID IN (SELECT ROWID
FROM SpatialIndex WHERE f_table_name='wayne_tracts'
AND search_frame=B.geometry)
AND B.placefp10 = '22000'
;
```

---

## Spatial Index

* Speeds performance
* Requires explicit inclusion in query
* Expand city boundary and search for tract centroids

```sql
select A.*
from wayne_tracts AS A, mi_places AS B
WHERE st_contains(st_buffer(B.geometry, 500), st_centroid(A.geometry))
AND A.ROWID IN (SELECT ROWID
	FROM SpatialIndex WHERE f_table_name='wayne_tracts'
	AND search_frame=B.geometry)
AND B.placefp10 = '22000';
```

---

## Detroit Homeownership


```sql
select A.*,
C.HD01_VD02 * 1.0 / C.HD01_VD01 * 100 AS own_rate
from wayne_tracts AS A, mi_places AS B
JOIN wayne_tenure AS C ON C.geoid2 = A.GEOID10
WHERE st_contains(st_buffer(B.geometry, 500), st_centroid(A.geometry))
AND A.ROWID IN (SELECT ROWID
	FROM SpatialIndex WHERE f_table_name='wayne_tracts'
	AND search_frame=B.geometry)
AND B.placefp10 = '22000';
```

---

## Making geometries from coordinates

* Assumes presence of fields name "LATITUDE" and "LONGITUDE"
* Assumes data were generated using NAD83 (EPSG:2898)

```bash
AddGeometryColumn('table', 'geometry', 4269, 'POINT', 'XY');
UPDATE table SET geometry=MakePoint(LONGITUDE,LATITUDE,4269);
SELECT CreateSpatialIndex('table', 'geometry');
```

---

## Transform SRS

* Project to US Albers Equal Area Conic
* ESRI:102003

```bash
AddGeometryColumn('table', 'geometry', 102003, 'POINT', 'XY');
UPDATE table SET geometry=ST_Transform(MakePoint(LONGITUDE,LATITUDE,4269), 102003);
SELECT CreateSpatialIndex('table', 'geometry');
```

---

## Spatialite & Python

* Use `pysqlite` module
* `conda install -c kbchoi pysqlite`

```python
from pysqlite2 import dbapi2 as sql

con = sql.connect("test.sqlite")
con.enable_load_extension(True)
con.execute("SELECT load_extension('mod_spatialite');")

```

---

## Use Case

```
For every city w/ pop. loss:
	- Select tracts where centroid within city
	- Calculate summary measures of loss
	- Pct of tracts that lost pop, etc.
	- Collect measures for each city for comparison 
```

---

## SQL for single case

```sql
SELECT C.geoid10, 
D.POP10, 
D.POP00,
C.aland10,
Hex(ST_AsBinary(C.geometry)) AS geometry /*convert to geopandas friendly format*/
FROM CenPop2010_Mean_TR AS A, us_place_2010 AS B 
JOIN tl_tracts_10 AS C ON A.tractlong = C.geoid10 /*join centroids to tract polys*/
JOIN tract_00_10 AS D ON C.geoid10 = D.GEOID 
WHERE ST_Contains(B.geometry, A.geometry) /* find pop. weighted centroids contained by city */
AND A.ROWID IN (
	SELECT ROWID FROM SpatialIndex
	WHERE f_table_name='CenPop2010_Mean_TR' AND search_frame=B.geometry)
AND B.geoid10 = '2965000' 	/*city being iterated over*/
AND D.POP00 >= 1; 		/*omit tracts without pop*/
```


---


## Geopandas for analysis

```python
df = gpd.GeoDataFrame.from_postgis(qry, con, geom_col='geometry', index_col='geoid10')

# calculate city-level loss
city_loss = ( df['POP10'].sum() - df['POP00'].sum() ) * 1.0 / df['POP00'].sum() * 100

# calculate pct change 2000 to 2010
df['ppchg_00-10'] = ( df['POP10'] - df['POP00'] ) * 1.0 / df['POP00'] * 100

# calculate pct of tracts that lost pop btw 2000 and 2010
extensiveness = len(df.loc[df['ppchg_00-10'] <= -5]) * 1.0 / len(df) * 100
```
---