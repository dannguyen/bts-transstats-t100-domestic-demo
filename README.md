# Data Diary for BTS Transtats, T100

This is a sample data diary that walks through the steps of acquiring [air carrier payload data from the Bureau of Transportation Statistics](http://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=259&DB_Short_Name=Air%20Carriers) and preparing it for SQL analysis.

## The steps

This guide will cover how to download a dataset about U.S. air carriers from a government webform, how to prune and clean the data, and how to insert it into a SQL database.


### Quick reference

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


## Data exploration

We will now __import__ the T100 data file into the SQL database of your choice: [Sequel Pro](http://www.sequelpro.com/) is a great GUI for Mac and MySQL, and [Firefox's SQLite Manager Plugin](https://addons.mozilla.org/en-US/firefox/addon/sqlite-manager/?src) is cross-platform.

For the purposes of this exercise, I use [Sequel Pro](http://www.sequelpro.com/)

1. __Data preparation__ - For some reason, the [T100 data file](data/sample-T100D-segment-data.csv) has a trailing column at the end of each row, which effectively creates an empty column. 

 ![Trailing commas in the T100 data](http://i.imgur.com/wanruuE.png)

 This is problematic in the __first row__ (i.e. the __headers__) during the import phase. A quick fix is to just type in some easy-to-ignore value. In the prepared file, I've simply named it `EMPTYFIELD`

2. __Create a database__ - In your database manager, create a database named `bts_data`

3. __Create a table__ - By default, most import managers will treat each field as a `VARCHAR` of length 255. Some of the fields in the data are clearly numbers (e.g. `DISTANCE`, `DEPARTURES_SCHEDULED`), and others have values much shorter than 255 (e.g. `ORIGIN_STATE_ABR`).

 We also want to __index__ some of the columns, such as `ORIGIN_AIRPORT_ID`, to speed up our queries.

 Here's the resulting SQL to create a table named `t100_domestic_carriers`:
 ~~~sql
 CREATE TABLE `t100_domestic_carriers` (
  `DEPARTURES_SCHEDULED` int(12) DEFAULT NULL,
  `DEPARTURES_PERFORMED` int(12) DEFAULT NULL,
  `PAYLOAD` int(11) DEFAULT NULL,
  `SEATS` int(11) DEFAULT NULL,
  `PASSENGERS` int(11) DEFAULT NULL,
  `FREIGHT` int(11) DEFAULT NULL,
  `MAIL` int(11) DEFAULT NULL,
  `DISTANCE` int(11) DEFAULT NULL,
  `RAMP_TO_RAMP` int(11) DEFAULT NULL,
  `AIR_TIME` int(11) DEFAULT NULL,
  `UNIQUE_CARRIER` char(6) DEFAULT NULL,
  `AIRLINE_ID` int(11) DEFAULT NULL,
  `UNIQUE_CARRIER_NAME` varchar(255) DEFAULT NULL,
  `UNIQUE_CARRIER_ENTITY` char(10) DEFAULT NULL,
  `REGION` char(2) DEFAULT NULL,
  `CARRIER` char(5) DEFAULT NULL,
  `CARRIER_NAME` varchar(255) DEFAULT NULL,
  `CARRIER_GROUP` char(2) DEFAULT NULL,
  `CARRIER_GROUP_NEW` char(2) DEFAULT NULL,
  `ORIGIN_AIRPORT_ID` char(12) DEFAULT NULL,
  `ORIGIN_AIRPORT_SEQ_ID` char(7) DEFAULT NULL,
  `ORIGIN_CITY_MARKET_ID` char(6) DEFAULT NULL,
  `ORIGIN` varchar(255) DEFAULT NULL,
  `ORIGIN_CITY_NAME` varchar(255) DEFAULT NULL,
  `ORIGIN_STATE_ABR` char(3) DEFAULT NULL,
  `ORIGIN_STATE_FIPS` varchar(255) DEFAULT NULL,
  `ORIGIN_STATE_NM` varchar(255) DEFAULT NULL,
  `ORIGIN_WAC` int(11) DEFAULT NULL,
  `DEST_AIRPORT_ID` char(12) DEFAULT NULL,
  `DEST_AIRPORT_SEQ_ID` char(7) DEFAULT NULL,
  `DEST_CITY_MARKET_ID` char(6) DEFAULT NULL,
  `DEST` varchar(255) DEFAULT NULL,
  `DEST_CITY_NAME` varchar(255) DEFAULT NULL,
  `DEST_STATE_ABR` char(3) DEFAULT NULL,
  `DEST_STATE_FIPS` varchar(255) DEFAULT NULL,
  `DEST_STATE_NM` varchar(255) DEFAULT NULL,
  `DEST_WAC` char(2) DEFAULT NULL,
  `AIRCRAFT_GROUP` char(3) DEFAULT NULL,
  `AIRCRAFT_TYPE` char(5) DEFAULT NULL,
  `AIRCRAFT_CONFIG` char(2) DEFAULT NULL,
  `YEAR` int(11) DEFAULT NULL,
  `QUARTER` int(11) DEFAULT NULL,
  `MONTH` int(11) DEFAULT NULL,
  `DISTANCE_GROUP` int(11) DEFAULT NULL,
  `CLASS` char(2) DEFAULT NULL,
  `EMPTYFIELD` char(1) DEFAULT NULL,
  KEY `UNIQUE_CARRIER` (`UNIQUE_CARRIER`),
  KEY `ORIGIN_AIRPORT_ID` (`ORIGIN_AIRPORT_ID`),
  KEY `ORIGIN_CITY_MARKET_ID` (`ORIGIN_CITY_MARKET_ID`),
  KEY `DEST_AIRPORT_ID` (`DEST_AIRPORT_ID`),
  KEY `DEST_CITY_MARKET_ID` (`DEST_CITY_MARKET_ID`),
  KEY `ORIGIN_AIRPORT_ID_2` (`ORIGIN_AIRPORT_ID`,`DEST_AIRPORT_ID`),
  KEY `ORIGIN_STATE_ABR` (`ORIGIN_STATE_ABR`),
  KEY `DEST_STATE_ABR` (`DEST_STATE_ABR`),
  KEY `YEAR_AND_MONTH` (`YEAR`,`MONTH`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
 ~~~

3. __Import the T100 CSV__ - Now simply import the CSV file into our newly created table. Sequel Pro will match up the columns in the CSV file to those in our `t100_domestic_carriers`

 ![Importing into Sequel Pro](http://i.imgur.com/OeKvLPO.png)








