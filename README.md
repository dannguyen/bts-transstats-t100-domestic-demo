# Data Diary for BTS Transtats, T100

This is a sample data diary that walks through the steps of acquiring [air carrier payload data from the Bureau of Transportation Statistics](http://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=259&DB_Short_Name=Air%20Carriers) and preparing it for SQL analysis.

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

## Download data file from BTS

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





