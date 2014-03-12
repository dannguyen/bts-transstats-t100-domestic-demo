# Data Diary for BTS Transtats, T100

This is a sample data diary that walks through the steps of acquiring [air carrier payload data from the Bureau of Transportation Statistics](http://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=259&DB_Short_Name=Air%20Carriers) and preparing it for SQL analysis.

## The steps

This guide will cover how to download a dataset about U.S. air carriers from a government webform, how to prune and clean the data, and how to insert it into a SQL database.


### Important links

- [Webform for T-100 Domestic Segment (U.S. Carriers)](http://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=259&DB_Short_Name=Air%20Carriers) - wherefrom the CSV is downloaded
- [Sequel Pro](http://www.sequelpro.com/) - A free MySQL client (for Mac OS X)
- [Firefox SQLite Manager](https://addons.mozilla.org/en-US/firefox/addon/sqlite-manager/?src) - A free plugin for SQLite (which will work just as well as MySQL)



## About the data

BTS has several databases related to air travel, including the T-100 Domestic Segment, which contains domestic flight route data. As of __March 2014__, the BTS has published the data through __November 2013__.

|            |                   |
| ---------  | ------------------|
| Records    |  __6.3+ million__ |
| Fields     | __45__ |
| First Year | __1990__ |
| Frequency  | __Monthly__ |

The [official data description](http://www.transtats.bts.gov/TableInfo.asp?Table_ID=259):

> This table contains domestic non-stop segment data reported by U.S. air carriers, including carrier, origin, destination, aircraft type and service class for transported passengers, freight and mail, available capacity, scheduled departures, departures performed, aircraft hours, and load factor when both origin and destination airports are located within the boundaries of the United States and its territories.

## Download air carrier data file from BTS

![The BTS web form](http://i.imgur.com/ppQw6Vx.png)

The data can be downloaded as flat CSV files [from this web form](http://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=259&DB_Short_Name=Air%20Carriers).

To download the __November 2013__ dataset, set the form options accordingly:

1. __Filter geography__ - `All`
2. __Filter year__ - `2013`
3. __Filter period__ - `November`
4. Check the __Select all fields__ checkbox
5. Then click the __Download__ button.

Your browser will download a __ZIP__ file weighing roughly 10.4 MB and it will be named something like: `932989999_T_T100D_SEGMENT_US_CARRIER_ONLY.zip`

Unzipping this file will produce a 94.1 MB plaintext CSV file. 

This is what the [first 200 rows of that CSV looks like](data/sample-T100D-segment-data.csv).

### Download lookup tables

The BTS provides several lookup tables that will be useful in keeping our Air Carrier data normalized. I've included them in this diary's `data/` directory for your convenience. You can download them from the BTS webform, though they annoyingly have the nonsensical file extension of `.csv-`

Table | Records | Description
------|--------:|------------
[L_AIRCRAFT_CONFIG.csv](data/lookup-tables/L_AIRCRAFT_CONFIG.csv)  |  6    | e.g. `"1","Passenger Configuration"` 
[L_AIRCRAFT_GROUP.csv](data/lookup-tables/L_AIRCRAFT_GROUP.csv)  |  10    |  e.g. `"7","Jet, 3-Engine"`
[L_AIRPORT_ID.csv](data/lookup-tables/L_AIRPORT_ID.csv)  |  6,260    |   a unique identifier for airports that persists regardless of changes to codes, e.g. `"10030","Port Vita, AK: Port Vita Airport"`
[L_AIRCRAFT_TYPE.csv](data/lookup-tables/L_AIRCRAFT_TYPE.csv)  |  385    |  e.g. `"889","B787-900 Dreamliner"`
[L_CITY_MARKET_ID.csv](data/lookup-tables/L_CITY_MARKET_ID.csv)  |  5,656    |   - a reference to a metro area that may be served by several airports., e.g. `"31703","New York City, NY (Metropolitan Area)"`
[L_REGION.csv](data/lookup-tables/L_REGION.csv)  |  6    |   e.g. `"D", "Domestic"`
[L_SERVICE_CLASS.csv](data/lookup-tables/L_SERVICE_CLASS.csv)  |  14    |  e.g.`"K","Scheduled Service K (F+G)"`
[L_UNIQUE_CARRIERS.csv](data/lookup-tables/L_UNIQUE_CARRIERS.csv)  |  1,565    |  e.g. `"DL","Delta Air Lines Inc."`
[L_WORLD_AREA_CODES.csv ](data/lookup-tables/L_WORLD_AREA_CODES.csv )  |  333    |   e.g. `"759","North Vietnam"`




