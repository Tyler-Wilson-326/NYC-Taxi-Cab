# Introduction  

This following project will be analyzing over 28 million rows of Green NYC taxicab data from 2017-2020. The data contains lots of useful information including pickup and drop-off date/time, location, trip distance, total amount paid to the driver and much more. In a bit a full breakdown of each table and the columns within will be provided to get a deeper insight and understanding of the data.

This project will be broken down into two parts; SQL data exploration answering some questions using queries as well as a Power BI component, creating a detailed dashboard showing interesting information and visuals pertaining to the data. This repo will only contain the SQL analysis, the dashboard will be provided in another repo.

# Credit

This data is sourced from the NYC Taxi and Limousine Commissioned and licensed in the public domain, the data was taken from maven analytics data playground and can be found [here]( https://www.mavenanalytics.io/data-playground?search=nyc)

To view the original source as well as additional information on taxis in NYC please visit the taxi  and limousine commission on the NYC government website  [here](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)

This contains data up until 2023, I used the maven analytics source, since the files provided were in CSV format opposed to the PARQUET format and they were appended yearly instead of monthly, making analysis easier. From my understanding there is no significant difference between the data provided from the original source and the source I have used from Maven Analytics, feel free to explore the original source yourself.

The original source also contains data for yellow taxis, which may be interesting to compare in the future comparing to the green taxi data.

# Background

In NYC there are two main types of taxis distinguished by their color and they are yellow and green taxis. Yellow taxis are your traditional taxis and are allowed to pick up passengers in all 5 boroughs. Green taxis were introduced in 2013 with the purpose of helping improve the availability of taxis in the NYC boroughs. They are allowed to pick up passengers above 110 St/E 96th St in Manhattan and in the remaining boroughs. Here is the map below, any area indicated by green is where yellow and green taxis can hail passengers. The yellow area indicates an area that only yellow taxis can pick up passengers.

![Boro-Taxi-4](https://github.com/Tyler-Wilson-326/NYC-Taxi-Trips/assets/141818698/9c75bebd-686e-46a1-907b-2f6101b7fc71)
[Source](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)

# Part 0 Data Cleaning and Transformation:

For the first part of this I will be cleaning and transforming the data slightly in order to help analyze the data easier. Firstly, the csv data is separated by year, I want to append them all into one table, we can use UNIONALL to achieve this in SQL, we need to first make sure though that the data is identical in all 4 tables, meaning same data types and same columns. One thing I noticed is that only in the 2019 and 2020 data, there is a congestion surcharge so we will need to add that column to the 2017 and 2018 data so we can append them.

### Adding Congestion Surcharge Fields

~~~~sql
ALTER TABLE trips_2017_taxi
ADD congestion_surcharge NUMERIC(10,2)

ALTER TABLE trips_2018_taxi
ADD congestion_surcharge NUMERIC(10,2)

UPDATE trips_2017_taxi

SET congestion_surcharge = 0

UPDATE trips_2018_taxi

SET congestion_surcharge = 0



~~~~

The specific scale and precision of the numeric field can be adjusted to how you see fit but i included up to ten digits and two decimal places, we also set this column value to 0 as there is no congestion surge data for these years, with that done we can now append the tables together.

### Union All Yearly Data Together into One Table

Below i append all the yearly data into one table, this will make it much easier to perform analysis on.
~~~~sql
CREATE TABLE taxi_data AS
SELECT * FROM trips_2017_taxi
UNION ALL
SELECT * FROM trips_2018_taxi
UNION ALL
SELECT * FROM trips_2019_taxi
UNION ALL
SELECT * FROM trips_2020_taxi;
~~~~
### Adjusting Data Types

Next, I want to update some column datatypes and add a couple new columns to make analysis a little easier when we start answering some queries.
First, I want to update the drop off time and pickup time columns to be of the timestamp format as opposed to its current variable character format. Next, I want to split this timestamp column to have just a date field and a time field.

~~~~sql
-- Changing data types
ALTER TABLE taxi_data
ALTER COLUMN pickup_time SET DATA TYPE TIMESTAMP 
USING pickup_time::timestamp WITHOUT time zone

ALTER TABLE taxi_data
ALTER COLUMN dropoff_time SET DATA TYPE TIMESTAMP 
USING dropoff_time::timestamp WITHOUT time zone

--Renaming original column to be more accurate
ALTER TABLE taxi_data

RENAME COLUMN pickup_time TO pickup_date_time

ALTER TABLE taxi_data

RENAME COLUMN dropoff_time TO dropoff_date_time

-- Adding new columns for date and time
ALTER TABLE taxi_data
ADD pickup_date date

ALTER TABLE taxi_data
ADD pickup_time time

ALTER TABLE taxi_data
ADD dropoff_date date

ALTER TABLE taxi_data
ADD dropoff_time time

-- inserting values into new columns

UPDATE taxi_data

SET pickup_date = date(pickup_date_time)


UPDATE taxi_data

SET dropoff_date = date(dropoff_date_time)


UPDATE taxi_data

SET pickup_time = pickup_date_time::time


UPDATE taxi_data

SET dropoff_time = dropoff_date_time::time

~~~~

# Table Structure

In our PostgreSQL Database we now have a total of 4 table a calender table, taxi zones table, 2017-2020 trip data table and a data dictionary. 

### Calender table

Below is a subset of what our calender table looks like in our database, we use this table to help join the pickup and dropoff time fields in our trip data table, this allows us to get more information such as the day name.

| date       | fiscal_year | fiscal_quarter | fiscal_month_number | fiscal_month_of_quarter | fiscal_week_of_year | day_of_week | fiscal_month_name | fiscal_month_year | fiscal_quarter_year | day_of_month_number | day_name  |
|------------|-------------|----------------|---------------------|-------------------------|---------------------|-------------|-------------------|-------------------|---------------------|---------------------|-----------|
| 2017-02-05 | 2017        | 1              | 1                   | 1                       | 1                   | 0           | February          | 17-Feb            | 12017               | 5                   | Sunday    |
| 2017-02-06 | 2017        | 1              | 1                   | 1                       | 1                   | 1           | February          | 17-Feb            | 12017               | 6                   | Monday    |
| 2017-02-07 | 2017        | 1              | 1                   | 1                       | 1                   | 2           | February          | 17-Feb            | 12017               | 7                   | Tuesday   |
| 2017-02-08 | 2017        | 1              | 1                   | 1                       | 1                   | 3           | February          | 17-Feb            | 12017               | 8                   | Wednesday |
| 2017-02-09 | 2017        | 1              | 1                   | 1                       | 1                   | 4           | February          | 17-Feb            | 12017               | 9                   | Thursday  |
| 2017-02-10 | 2017        | 1              | 1                   | 1                       | 1                   | 5           | February          | 17-Feb            | 12017               | 10                  | Friday    |
| 2017-02-11 | 2017        | 1              | 1                   | 1                       | 1                   | 6           | February          | 17-Feb            | 12017               | 11                  | Saturday  |
| 2017-02-12 | 2017        | 1              | 1                   | 1                       | 2                   | 0           | February          | 17-Feb            | 12017               | 12                  | Sunday    |
| 2017-02-13 | 2017        | 1              | 1                   | 1                       | 2                   | 1           | February          | 17-Feb            | 12017               | 13                  | Monday    |
| 2017-02-14 | 2017        | 1              | 1                   | 1                       | 2                   | 2           | February          | 17-Feb            | 12017               | 14                  | Tuesday   |

### Taxi Zones table

This table is a subset showing more information about the locations within NYC, this table shows the specific location zone name, the borough name and the corresponding location ID field which we can use to join against out pickup or drop off location ID in our trip data table

| location_id | borough       | zone                    | service_zone |
|-------------|---------------|-------------------------|--------------|
| 1           | EWR           | Newark Airport          | EWR          |
| 2           | Queens        | Jamaica Bay             | Boro Zone    |
| 3           | Bronx         | Allerton/Pelham Gardens | Boro Zone    |
| 4           | Manhattan     | Alphabet City           | Yellow Zone  |
| 5           | Staten Island | Arden Heights           | Boro Zone    |
| 6           | Staten Island | Arrochar/Fort Wadsworth | Boro Zone    |
| 7           | Queens        | Astoria                 | Boro Zone    |
| 8           | Queens        | Astoria Park            | Boro Zone    |
| 9           | Queens        | Auburndale              | Boro Zone    |
| 10          | Queens        | Baisley Park            | Boro Zone    |



### Trip Data
Now moving onto our trip data, remeber this is all 4 years appended and in total we have around 27 million trips recorded over the 4 years, below is just a subset of data: following this table will be a data dictionary table describing what each column means, we do have some new columns though that we made earlier which were the respective data and time fields for dropoff and pickup.

| vendor_id | pickup_date_time    | dropoff_date_time   | store_and_fwd_flag | ratecode_id | pu_location_id | do_location_id | passenger_count | trip_distance | fare_amount | extra | mta_tax | tip_amount | tolls_amount | improvement_surcharge | total_amount | payment_type | trip_type | congestion_surcharge | pickup_date | pickup_time | dropoff_date | dropoff_time |
|-----------|---------------------|---------------------|--------------------|-------------|----------------|----------------|-----------------|---------------|-------------|-------|---------|------------|--------------|-----------------------|--------------|--------------|-----------|----------------------|-------------|-------------|--------------|--------------|
| 2         | 2017-04-29 11:05:47 | 2017-04-29 11:06:31 | N                  | 1           | 193            | 193            | 5               | 0.00          | 0.00        | 0.00  | 0.00    | 0.00       | 0.00         | 0.00                  | 0.00         | 1            | 1         | 0.00                 | 2017-04-29  | 11:05:47    | 2017-04-29   | 11:06:31     |
| 1         | 2019-01-06 20:18:14 | 2019-01-06 20:18:36 | Y                  | 5           | 70             | 70             | 3               | 0.00          | 0.00        | 0.00  | 0.00    | 0.00       | 0.00         | 0.00                  | 0.00         | 2            | 2         | NULL                 | 2019-01-06  | 20:18:14    | 2019-01-06   | 20:18:36     |
| 2         | 2019-01-11 17:15:52 | 2019-01-11 17:20:40 | N                  | 1           | 95             | 95             | 3               | 0.53          | 5.00        | 1.00  | 0.50    | 1.00       | 0.00         | 0.30                  | 7.80         | 1            | 1         | NULL                 | 2019-01-11  | 17:15:52    | 2019-01-11   | 17:20:40     |
| 1         | 2019-01-01 02:34:51 | 2019-01-01 02:39:15 | N                  | 1           | 256            | 255            | 2               | 0.70          | 5.00        | 0.50  | 0.50    | 2.00       | 0.00         | 0.30                  | 8.30         | 1            | 1         | NULL                 | 2019-01-01  | 02:34:51    | 2019-01-01   | 02:39:15     |
| 2         | 2019-01-09 19:23:47 | 2019-01-09 19:42:42 | N                  | 1           | 97             | 67             | 1               | 7.00          | 22.00       | 1.00  | 0.50    | 5.20       | 0.00         | 0.30                  | 29.00        | 1            | 1         | NULL                 | 2019-01-09  | 19:23:47    | 2019-01-09   | 19:42:42     |
| 2         | 2019-01-18 17:52:21 | 2019-01-18 18:03:21 | N                  | 1           | 226            | 129            | 1               | 2.00          | 9.50        | 1.00  | 0.50    | 0.70       | 0.00         | 0.30                  | 12.00        | 1            | 1         | NULL                 | 2019-01-18  | 17:52:21    | 2019-01-18   | 18:03:21     |
| 1         | 2017-01-01 09:00:01 | 2017-01-01 09:03:56 | N                  | 1           | 74             | 41             | 1               | 0.70          | 5.00        | 0.00  | 0.50    | 0.00       | 0.00         | 0.30                  | 5.80         | 2            | 1         | 0.00                 | 2017-01-01  | 09:00:01    | 2017-01-01   | 09:03:56     |
| 1         | 2017-01-01 18:57:55 | 2017-01-01 19:01:16 | N                  | 1           | 42             | 41             | 1               | 0.70          | 5.00        | 0.00  | 0.50    | 0.00       | 0.00         | 0.30                  | 5.80         | 2            | 1         | 0.00                 | 2017-01-01  | 18:57:55    | 2017-01-01   | 19:01:16     |
| 2         | 2017-01-02 06:55:47 | 2017-01-02 06:58:54 | N                  | 1           | 42             | 41             | 1               | 0.70          | 5.00        | 0.00  | 0.50    | 0.00       | 0.00         | 0.30                  | 5.80         | 2            | 1         | 0.00                 | 2017-01-02  | 06:55:47    | 2017-01-02   | 06:58:54     |
| 2         | 2017-01-02 14:34:17 | 2017-01-02 14:39:29 | N                  | 1           | 74             | 41             | 1               | 0.70          | 5.00        | 0.00  | 0.50    | 0.00       | 0.00         | 0.30                  | 5.80         | 2            | 1         | 0.00                 | 2017-01-02  | 14:34:17    | 2017-01-02   | 14:39:29     |

###  Data Dictionary For the Taxi Data

Here is the data dictionary describing each field in the taxi trip data table.

| field                 | description                                                                                                                                                                                                                                          |
|-----------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| VendorID              | A code indicating the LPEP provider that provided the record (1= Creative Mobile Technologies, LLC; 2= Verifone Inc.)                                                                                                                                |
| lpep_pickup_datetime  | The date and time when the meter was engaged                                                                                                                                                                                                         |
| lpep_dropoff_datetime | The date and time when the meter was disengaged                                                                                                                                                                                                      |
| store_and_fwd_flag    | This flag indicates whether the trip record was held in vehicle memory before sending to the vendor, aka store and forward, because the vehicle did not have a connection to the server (Y= store and forward trip; N= not a store and forward trip) |
| RatecodeID            | The final rate code in effect at the end of the trip (1= Standard rate; 2= JFK; 3= Newark; 4= Nassau or Westchester; 5= Negotiated fare; 6= Group ride)                                                                                              |
| PULocationID          | TLC Taxi Zone in which the taximeter was engaged                                                                                                                                                                                                     |
| DOLocationID          | TLC Taxi Zone in which the taximeter was disengaged                                                                                                                                                                                                  |
| passenger_count       | The number of passengers in the vehicle (this is a driver entered value)                                                                                                                                                                             |
| trip_distance         | The elapsed trip distance in miles reported by the taximeter                                                                                                                                                                                         |
| fare_amount           | The time-and-distance fare calculated by the meter                                                                                                                                                                                                   |
| extra                 | Miscellaneous extras and surcharges (this only includes the $0.50 and $1 rush hour and overnight charges)                                                                                                                                            |
| mta_tax               | $0.50 MTA tax that is automatically triggered based on the metered rate in use                                                                                                                                                                       |
| tip_amount            | Tip amount (automatically populated for credit card tips - cash tips are not included)                                                                                                                                                               |
| tolls_amount          | Total amount of all tolls paid in trip                                                                                                                                                                                                               |
| improvement_surcharge | $0.30 improvement surcharge assessed on hailed trips at the flag drop                                                                                                                                                                                |
| total_amount          | The total amount charged to passengers (does not include cash tips)                                                                                                                                                                                  |
| payment_type          | A numeric code signifying how the passenger paid for the trip (1= Credit card; 2= Cash; 3= No charge; 4= Dispute; 5= Unknown; 6= Voided trip)                                                                                                        |
| trip_type             | A code indicating whether the trip was a street-hail or a dispatch that is automatically assigned based on the metered rate in use but can be altered by the driver (1= Street-hail; 2= Dispatch)                                                    |
| congestion_surcharge  | Congestion surcharge for trips that start, end or pass through the congestion zone in Manhattan, south of 96th street ($2.50 for non-shared trips in Yellow Taxis; $2.75 for non-shared trips in Green Taxis)                                        |



# EDA and Further Data Cleaning

Next let’s explore the data to see if there is crazy outliers that make like no sense for example negative distance, negative surcharges, values that are way to high etc. Note Python would probably be a lot more powerful for any EDA due to its powerful libraries and quick visualizations with pandas and matplotlib, however I wanted to structure this project using entirely SQL and Power BI so I will complete these projects using solely those two tools. I will provide my reasoning for any changes and make my best judgment in order to make the analysis more accurate.

**NOTE:** This data cleaning process was done alongside working on the questions found later in this document and organized here all together, what this means is that depening on the order you apply these cleaning steps, you may have slightly different results. However the process to delete and identify remains the same. 

### Investigating the Vendor ID Field

First thing I want to do is check how many unique vendor IDs are there? The data dictionary indicates there should only be two which are as follows: 

1= Creative Mobile Technologies, LLC; 2= VeriFone Inc.


~~~~sql
SELECT vendor_id, COUNT(*) AS num FROM taxi_data
GROUP BY vendor_id
~~~~

| vendor_id | num      |
|-----------|----------|
| 1         | 4814128  |
| 2         | 22569744 |
| NULL      | 942199   |

So here we can see we have some NULL values, 942 199 which is almost a million values, I will be dropping these values since this indicates there is likely some error with the way the data was collected. Now I could just keep these in, but I am unsure of the accuracy of the data and will be dropping them as the documentation only indicates 2 different vendor IDs should be present. 

~~~~sql
DELETE FROM taxi_data
WHERE vendor_id IS NULL;
~~~~

### Unknown Locations

One thing I want to look at now is specific to drop off and pickup locations, we have two ID’s 264 and 265 which indicate unknown locations, and I don’t believe will be useful for analysis as I’m not sure where these places indicate so I will remove them as such, if more information was provided as to what these IDs referred to, then I would keep them in. In a real-life scenario this is something that should be communicated with whoever provided the data.
~~~~sql
-- next lets look at the unknown locations 

SELECT COUNT(*) AS num_loc FROM taxi_data

WHERE pu_location_id = 264 OR pu_location_id = 265 OR do_location_id = 264 OR do_location_id = 265 
~~~~

| num_loc |
|---------|
| 135545  |

Here we can see we have 135545 entries containing these unknown locations as we will remove them.

~~~~sql
-- lets remove these rows now

DELETE FROM taxi_data
WHERE pu_location_id = 264 OR pu_location_id = 265 OR do_location_id = 264 OR do_location_id = 265
~~~~ 

### Investigating the Distance Field

Next lets look at the distance column, this is an easy contender for having weird values such as negative values or very large unrealistic values. First the below query looks at if there are any negative values, here we see there are and therefore we will remove them. 

~~~~sql
SELECT (trip_distance), COUNT(*) FROM taxi_data
WHERE (trip_distance) < 0
GROUP BY (trip_distance)
~~~~

| neg_val |
|---------|
| 61   |

Now we want to look at other distance values that would not make sense such as unreasonably large distances. . NYC is 302.5 miles squared. The longest possible distance to travel would be from bottom of Staten Island (Tottenville) to the northernmost point of the Bronx (Wakefield) and could be anywhere between 40-60 miles based on detours, road conditions etc. Now it is very unrealistic to take a taxicab that distance in the first place, but it is still possible and there is a chance that has happened in this data set over the course of the 4 years. Let’s explore the data and see how many values we have greater than 60 miles.

~~~~sql
SELECT pickup_date_time,dropoff_date_time,pu_location_id,do_location_id,trip_distance FROM taxi_data
WHERE ROUND(trip_distance,0) > 60
ORDER BY trip_distance ASC
~~~~

| dist   | count |
|--------|-------|
| 134122 | 1     |
| 102497 | 1     |
| 35757  | 1     |
| 25732  | 1     |
| 24430  | 1     |
| 12468  | 1     |
| 8006   | 1     |
| 6726   | 1     |
| 667    | 7     |
| 640    | 1     |
| 621    | 1     |
| 604    | 1     |
| 483    | 1     |
| 333    | 1     |
| 220    | 1     |
| 202    | 1     |
| 184    | 3     |
| 168    | 1     |
| 159    | 1     |
| 152    | 1     |
| 151    | 1     |
| 142    | 1     |
| 138    | 1     |
| 131    | 1     |
| 130    | 2     |
| 129    | 1     |
| 128    | 1     |
| 127    | 1     |
| 124    | 2     |
| 120    | 4     |
| 119    | 1     |
| 117    | 1     |
| 113    | 1     |
| 109    | 1     |
| 108    | 1     |
| 106    | 2     |
| 105    | 1     |
| 104    | 3     |
| 103    | 2     |
| 102    | 2     |
| 101    | 2     |
| 100    | 2     |
| 99     | 1     |
| 98     | 1     |
| 96     | 1     |
| 94     | 1     |
| 91     | 2     |
| 90     | 2     |
| 89     | 3     |
| 88     | 1     |
| 87     | 4     |
| 86     | 1     |
| 85     | 3     |
| 84     | 3     |
| 83     | 3     |
| 82     | 4     |
| 81     | 2     |
| 80     | 4     |
| 78     | 3     |
| 77     | 3     |
| 76     | 1     |
| 75     | 1     |
| 74     | 2     |
| 73     | 9     |
| 72     | 4     |
| 71     | 4     |
| 70     | 9     |
| 69     | 9     |
| 68     | 5     |
| 67     | 7     |
| 66     | 14    |
| 65     | 10    |
| 64     | 6     |
| 63     | 6     |
| 62     | 10    |
| 61     | 8     |

Here we can see a fair bit of unrealistic distances, lets get a count of how many trips are above 60 miles

~~~~sql
SELECT COUNT(*) AS num_trips FROM taxi_data
WHERE trip_distance > 60
~~~~
| num_trips |
|---------|
| 209  |

We have a total of 209 trips above 60 miles, looking at some of the values present comparing pickup-time to drop-off-time we can see that it does make sense for some values being this high but most of it does not and we have no way of confirming if a 60 mile + trip for example is accurate, realistically it is not but it could be possible. I will be removing any values greater than 60 and keeping everything below it, it’s not perfect but I think doing this cleans the data better than leaving as is. It is only 209 trips out of millions of trips, so it is quite negligible on the overall data set

~~~~sql
DELETE FROM taxi_data
WHERE trip_distance > 60
~~~~

### Negative Payments
Next let’s look at the total amount charged and again look for any unrealistic values, starting with negative values. 

~~~~sql
SELECT COUNT(total_amount) AS neg_val FROM taxi_data
WHERE total_amount < 0
~~~~

| num_trips |
|---------|
| 71 507   |

We can see here we have a total of 71 507 trips that had a negative amount charged. There could maybe be multiple reasons for this. First there could be refunds based on the service and a negative value indicates a refund applied to the customer, no way of knowing whether this is true. Two some sort of system malfunction where the value is actually just the positive amount but instead displayed a negative value. There is just no way to know the reasons for these values, so I will remove any negative values because realistically that is not something that should be in the data, and we have no proof or justification to keep it in.

### Investigating the Passenger Column 

Now let us investigate the passenger column to see if we see any unusual information.

~~~~sql
SELECT passenger_count, COUNT(*) FROM taxi_data
GROUP by passenger_count
~~~~

| passenger_count | count    |
|-----------------|----------|
| 0               | 28271    |
| 1               | 23071230 |
| 2               | 2098823  |
| 3               | 451119   |
| 4               | 155278   |
| 5               | 903922   |
| 6               | 466755   |
| 7               | 477      |
| 8               | 521      |
| 9               | 150      |


Here we can see passengers ranging from 0-9 people, you can’t have a taxi trip without a passenger so those rows with 0 will be dropped for sure. There is a chance this might be canceled trips but again for simplicity we will be just dropping those entirely because there isn’t a field that indicates if a trip was cancelled or not. According to the Driver Rule 54-15(g) Chapter 54 - Drivers of Taxicabs and Street Hail Liveries document found [here](https://www.nyc.gov/assets/tlc/downloads/pdf/rule_book_current_chapter_54.pdf) a maximum of 5 passengers is allowed with a 6th passenger being allowed if the passenger is under 7 so I will remove 7-9 in terms of passenger counts leaving only data from 1-6 passengers.

~~~~sql
DELETE FROM taxi_data
WHERE passenger_count = 0 OR passenger_count > 6
~~~~
### Rate Code ID Investigation

Next let’s look at rate code id to see if we have any weird values here:

~~~~sql
SELECT ratecode_id, COUNT(*) AS num FROM taxi_data
GROUP BY ratecode_id
~~~~

| ratecode_id | num      |
|-------------|----------|
| 1           | 26380200 |
| 2           | 53159    |
| 3           | 12023    |
| 4           | 4043     |
| 5           | 693827   |
| 6           | 320      |
| 99           | 28      |

Here we can see the typically rate codes from 1-6 but we also see this unknown rate code of 99, looking at the data dictionary there is nothing that indicates this rate code, we also see it’s a very small amount of 28 and therefore I think its safe to remove from the data as we have no official documentation supporting what this value is and its quite insignificant in size to the overall data set.

~~~~sql
DELETE FROM taxi_data
WHERE ratecode_id = 99
~~~~

### Investigating the Date Fields

Now let’s look at our data by year, this set should only contain data from years 2017-2020 so let’s examine this now.

~~~~sql
SELECT EXTRACT(YEAR FROM pickup_date) AS YEAR, COUNT(*) as num_trips FROM taxi_data
GROUP BY 1
ORDER BY 1 ASC
~~~~

| year | num_trips |
|------|-----------|
| 2008 | 96        |
| 2009 | 273       |
| 2010 | 329       |
| 2012 | 3         |
| 2017 | 11663400  |
| 2018 | 8738032   |
| 2019 | 5557759   |
| 2020 | 1187201   |
| 2021 | 1         |
| 2030 | 2         |
| 2035 | 1         |
| 2041 | 1         |
| 2081 | 1         |

We can see some errors here as we have dates entered less then 2017 and greater than 2020, we will therefore be dropping them as these dates should not be included in this dataset as this dataset should only contain dates between 2017-2020.

~~~~sql
DELETE FROM taxi_data
WHERE EXTRACT(YEAR FROM pickup_date) > 2020  OR EXTRACT(YEAR FROM pickup_date) < 2017 
~~~~

Another thing we need to check is we need to make sure that we don’t have any dropoff date times that are less than the pickup date times as that doesn’t make sense

~~~~sql
SELECT COUNT(*) FROM taxi_data
WHERE dropoff_date_time < pickup_date_time
~~~~

| count |
|-------|
| 97    |

Here we can see that there are 97 counts at the moment of this cleaning that have drop-off dates less than the pickups dates, so we will drop these.

~~~~sql
DELETE FROM taxi_data
WHERE dropoff_date_time < pickup_date_time
~~~~

Finally let’s see if there is any trips that have the exact same date time for pickup and drop-off

~~~~sql
SELECT COUNT(*) from taxi_data
WHERE dropoff_date_time = pickup_date_time
~~~~

| count |
|-------|
| 2723  |

Here we see we have 2723 trips that had the exact same date and time, this is not possible and likely is a system error/data collection error, since the meter itself has to be engaged by the taxicab driver and therefore I will remove these entries as they do not make much sense, it is also a relatively insignificant portion of our data set.

~~~~sql
DELETE FROM taxi_data
WHERE dropoff_date_time = pickup_date_time
~~~~

With that done I think we are done with our  cleaning and can move on to answering some questions. Remember this cleaning process was updated as i ran into issues analyzing the data set, so performing these exact same cleaning steps in order may not produce the exact same results as what i have here.

# Part 1: SQL Data Exploration

The first part of this project will be to create some SQL queries, to answer some questions that may be useful for analysis. All data was imported into PostgreSQL, using PGAdmin4. Below are 10 questions that I will initially answer, more may be added in the future as I explore the data more.

Question list:

### **1. Busiest Taxi Zones:** Which taxi zones have the highest number of pickups and drop-offs?

For this question we can look specifically at the actual zone and the borough itself, lets start with the borough and then look at the specific taxi zone. Below shows a count of location pickups and drop offs as well as a percentage that finds the total percentage of each borough of the whole for both pickups and drop offs.

~~~~sql
(SELECT p.borough, 
SUM(CASE WHEN p.location_id = taxi_data.pu_location_id THEN 1 ELSE 0 END) AS pickups,
SUM(CASE WHEN p.location_id = taxi_data.do_location_id THEN 1 ELSE 0 END) AS dropoffs, 
ROUND((COUNT(*)/ SUM(COUNT(*)) OVER())*100,2) AS percentage

FROM taxi_data 

JOIN taxi_zones p ON p.location_id = taxi_data.pu_location_id 
OR p.location_id = taxi_data.do_location_id

GROUP BY p.borough
ORDER BY pickups DESC)
~~~~
| borough       | pickups | dropoffs | percentage |
|---------------|---------|----------|------------|
| Manhattan     | 9120475 | 10238189 | 36.16      |
| Brooklyn      | 8673913 | 7328282  | 29.89      |
| Queens        | 7972989 | 7791860  | 27.99      |
| Bronx         | 1367800 | 1761621  | 5.91       |
| Staten Island | 7952    | 14269    | 0.04       |
| EWR           | 443     | 9351     | 0.02       |

we can see here that Manhattan ranks the highest for both drop offs and pickups, logically this makes sense as Manhattan is home to some of the most popular tourist attractions like Times Square and the Empire State building, making it a prime candidate for taxi rides.

Next lets look at the most popular zones, here we will look at the top ten zones. First lets look at pickup zones specifically

~~~~sql
WITH CTE AS (SELECT COUNT(*) AS pickups, zone, DENSE_RANK() OVER(ORDER BY COUNT(*) DESC) AS rank, ROUND((COUNT(*)/ SUM(COUNT(*)) OVER())*100,2) AS percentage  FROM taxi_data

JOIN taxi_zones ON taxi_zones.location_id = taxi_data.pu_location_id
GROUP BY zone
ORDER BY pickups DESC)

SELECT pickups, zone, percentage, rank FROM cte 

WHERE rank <= 10
~~~~

| zone                      | pickups | percentage | rank |
|---------------------------|---------|------------|------|
| East Harlem North         | 1865086 | 6.87       | 1    |
| East Harlem South         | 1608984 | 5.93       | 2    |
| Central Harlem            | 1568175 | 5.78       | 3    |
| Astoria                   | 1288281 | 4.75       | 4    |
| Elmhurst                  | 1194505 | 4.40       | 5    |
| Morningside Heights       | 1085597 | 4.00       | 6    |
| Central Harlem North      | 962577  | 3.55       | 7    |
| Fort Greene               | 815678  | 3.01       | 8    |
| Forest Hills              | 751366  | 2.77       | 9    |
| Williamsburg (North Side) | 750895  | 2.77       | 10   |

Next lets look at the most popular zones for dropoff locations.

~~~~sql
WITH CTE AS (SELECT COUNT(*) AS dropoffs, zone, DENSE_RANK() OVER(ORDER BY COUNT(*) DESC) AS rank, ROUND((COUNT(*)/ SUM(COUNT(*)) OVER())*100,2) AS percentage  FROM taxi_data

JOIN taxi_zones ON taxi_zones.location_id = taxi_data.do_location_id
GROUP BY zone
ORDER BY dropoffs DESC)

SELECT dropoffs, zone,percentage, rank FROM cte 

WHERE rank <= 10
~~~~

| zone                 | dropoffs | percentage | rank |
|----------------------|----------|------------|------|
| East Harlem North    | 978184   | 3.60       | 1    |
| Central Harlem North | 942823   | 3.47       | 2    |
| Central Harlem       | 844149   | 3.11       | 3    |
| Astoria              | 807477   | 2.97       | 4    |
| Jackson Heights      | 742920   | 2.74       | 5    |
| East Harlem South    | 659803   | 2.43       | 6    |
| Park Slope           | 592139   | 2.18       | 7    |
| Morningside Heights  | 541216   | 1.99       | 8    |
| Elmhurst             | 507254   | 1.87       | 9    |
| Steinway             | 471664   | 1.74       | 10   |

Here i use window functions instead of just using limit 10  order by count because the issue with limit 10 is that it cuts off the first 10 results, if there is a tie at tenth place for example it wont show both. Its unlikely that there would be ties since there is 200+ zones but as a best practice I like to use rank windows functions instead of limiting the output.

It is also interesting to see the difference between pickups and dropoffs here we can see that the top locations for pickups have a lot more total trips then the top drop off locations meaning pickups are more top sided, look at the percentage distribution.

### **2. Most Popular Payment Method:** What is the most common payment type for taxi trips?

For this question I am expecting to see credit card and cash as the most popular payment method as the other options such as unknown, dispute etc. would be  uncommon overall.
~~~~sql
SELECT CASE 
WHEN payment_type = 1 THEN 'Credit Card' 
WHEN payment_type = 2 THEN 'Cash' 
WHEN payment_type = 3 THEN 'No Charge' 
WHEN payment_type = 4 THEN 'Dispute' 
WHEN payment_type = 5 THEN 'Unknown' 
WHEN payment_type = 6 THEN 'Voided' 

ELSE 'N/A' END AS payment,


COUNT(*) AS num,   ROUND((COUNT(*)/ SUM(COUNT(*)) OVER())*100,2) AS percentage  FROM taxi_data

GROUP BY 1

ORDER BY num DESC
~~~~


| payment     | num      | percentage |
|-------------|----------|------------|
| Credit Card | 14704430 | 54.17      |
| Cash        | 12315584 | 45.37      |
| No Charge   | 79491    | 0.29       |
| Dispute     | 42916    | 0.16       |
| Unknown     | 1151     | 0.00       |

Here we look at distribution of payment type seeing that credit card and cash accounting for about 99% of total trips as expected.

### **3. Peak Hours:** During which hours of the day do taxis experience the highest demand?
here we can look at the distribution of pickups and drop offs for each hour of the day, below is a query showing that.
~~~~sql
WITH CTE AS (SELECT EXTRACT(HOUR FROM pickup_time) AS hour, count(*) as pickup_count, 0 as dropoff_count
FROM taxi_data 
GROUP BY 1
UNION ALL

SELECT EXTRACT(HOUR FROM dropoff_time) AS hour, 0 AS pickup_count, COUNT(*) AS dropoff_count
FROM taxi_data
GROUP BY 1)

SELECT hour, SUM(pickup_count) AS total_pickups, SUM(dropoff_count) AS total_dropoffs FROM CTE
GROUP BY hour
ORDER BY total_pickups DESC
~~~~
| hour | total_pickups | total_dropoffs |
|------|---------------|----------------|
| 18   | 1919311       | 1961322        |
| 17   | 1808685       | 1784803        |
| 19   | 1778651       | 1863413        |
| 16   | 1675216       | 1639213        |
| 15   | 1555612       | 1505149        |
| 20   | 1519990       | 1572520        |
| 14   | 1404732       | 1347907        |
| 21   | 1394905       | 1412573        |
| 9    | 1301241       | 1315830        |
| 22   | 1269931       | 1292915        |
| 12   | 1259021       | 1259368        |
| 13   | 1257769       | 1249065        |
| 10   | 1257374       | 1279577        |
| 11   | 1248249       | 1241845        |
| 8    | 1200153       | 1111027        |
| 23   | 1125930       | 1163276        |
| 0    | 891514        | 977720         |
| 7    | 798448        | 673485         |
| 1    | 663345        | 702920         |
| 2    | 475552        | 501767         |
| 6    | 395458        | 327124         |
| 3    | 376022        | 381917         |
| 4    | 322556        | 339670         |
| 5    | 243907        | 239166         |

We can see here that typically hours between late afternoon and early evening have the most trips and this makes perfect sense considering this time would be considered rush hour where people need to get home from work and also times when tourists would finish their activities for the day and head back to their hotels. Later when we create a dashboard, we can visualize this by day of the week and see if there are any changes of hours there.



### **4. Average Trip Distance:** What is the average trip distance for each rate code?
Below is a query finding the average distance for each rate code.
~~~~sql
SELECT CASE 
WHEN ratecode_id = 1 THEN 'Standard Rate' 
WHEN ratecode_id = 2 THEN 'JFK' 
WHEN ratecode_id = 3 THEN 'Newark' 
WHEN ratecode_id = 4 THEN 'Nassau or Westchester' 
WHEN ratecode_id = 5 THEN 'Negotiated fare' 
WHEN ratecode_id = 6 THEN 'Group ride' 

ELSE 'Unknown' END AS rate_code, ROUND(AVG(trip_distance)*1.60934,2) AS avg_trip_distance_km FROM taxi_data

GROUP BY 1

ORDER BY avg_trip_distance_km DESC
~~~~
| rate_code             | avg_trip_distance_km |
|-----------------------|----------------------|
| Nassau or Westchester | 22.44                |
| JFK                   | 20.15                |
| Newark                | 19.87                |
| Negotiated fare       | 8.69                 |
| Standard Rate         | 4.54                 |
| Group ride            | 2.31                 |

We can see here on average Westchester and Nassau to be the largest distance wise; this makes sense considering these counties are outside NYC and therefore on average the distance traveled from boroughs within NYC would be farther. Next, we have JFK and Newark which are airports. Newark being outside NYC in New Jersey and JFK being at the very south end of NYC in the Queen’s Borough. We then have the remaining fares that are not location specified at much smaller average distance values. All these numbers make complete sense considering the reasoning above.

### **5. Earnings Analysis:** How do the earnings (total amount charged to passengers) vary across different days of the week?

Below is a query showing the total amount charged to passengers by day of the week. I am expecting to likely see the weekend having the largest amount of earnings compared to weekends. Here I am joining the date to the drop off date as that is when you would typically pay a taxi instead of the pickup date.

~~~~sql
SELECT SUM(total_amount) AS total_amount, COUNT(*) AS num_trips, day_name FROM taxi_data
JOIN calender ON taxi_data.dropoff_date = calender.date
GROUP BY day_name
ORDER BY total_amount DESC
~~~~

| day_name  | total_amount | num_trips |
|-----------|--------------|-----------|
| Friday    | 62802821.39  | 4113341   |
| Saturday  | 62056039.77  | 4193234   |
| Thursday  | 58024185.91  | 3753321   |
| Wednesday | 55350415.55  | 3594600   |
| Sunday    | 52739428.14  | 3532729   |
| Tuesday   | 52612201.14  | 3430205   |
| Monday    | 50077873.17  | 3304380   |

As expected we can see the total amount of earnings at its highest during the weekends, the reason is weekends would be the day off work where tourists or other people in NYC can engage in fun activities such as visiting landmarks, going to movie theatres, shopping districts also going to bars/clubs etc. going to bars on the weekend seems a more common choice then during the weekdays when people may go to work, considering you cannot drive yourself when you drink this would also cause a lot more taxis during the nighttime on these days. Again, the weekdays will also have people go to work potentially using taxis, but I think the leisurely activities will make more use of taxis. 


Also in a general sense the more trips there are the more earnings there is, We do see here though that despite having more trips on Saturday, it has earned less than Friday but it is not by a worrying amount 

I am also interested to look at the evening  hours on Saturday/Friday compared to the rest of the week to see if what I said earlier above NYC night life makes sense.

~~~~sql
WITH CTE AS (SELECT day_name, CASE WHEN EXTRACT(HOUR FROM dropoff_time) IN (22,23,0,1,2,3) THEN 1.0 else 0.0 END AS night_trips,

CASE WHEN EXTRACT(HOUR FROM dropoff_time) NOT IN (22,23,0,1,2,3) THEN 1.0 else 0.0 END AS day_trips

FROM taxi_data

JOIN calender ON taxi_data.dropoff_date = calender.date)

SELECT day_name, SUM(night_trips) AS night_trips, SUM(day_trips) AS day_trips, ROUND((SUM(night_trips)/COUNT(*)) * 100.0,2) as night_life_percentage FROM CTE
GROUP BY day_name
ORDER BY night_trips DESC
~~~~

| day_name  | night_trips | day_trips | night_life_percentage |
|-----------|-------------|-----------|-----------------------|
| Saturday  | 1086323.0   | 3106911.0 | 25.91                 |
| Sunday    | 954431.0    | 2578298.0 | 27.02                 |
| Friday    | 750478.0    | 3362863.0 | 18.24                 |
| Thursday  | 567973.0    | 3185348.0 | 15.13                 |
| Wednesday | 499173.0    | 3095427.0 | 13.89                 |
| Monday    | 454656.0    | 2849724.0 | 13.76                 |
| Tuesday   | 446807.0    | 2983398.0 | 13.03                 |

Here we have some interesting results we see as expected Saturday to be at top as this is typically the best day to go out as people are usually off work that day, however we see a lot of Sunday trips actually surpassing Friday. This is also interesting because Sunday has fewer total trips then Friday does. Sunday night trips account for 27% of its total trips! I Have used a time of 10:00 pm - 3:00 am to classify as night trips, this time can be adjusted based on what people would consider the time for “night life”. As expected, though Monday-Thursday has much less night trips comparatively to total trips. What could be the reason for this? Well, some potential reasons include that Sunday can still be considered a weekend, it also can be a popular day for tourist activities, so people who wouldn’t have work the Monday after. Again, just an interesting insight as this was not what I initially expected but it isn’t completely unreasonable like a day such as Thursday having the highest share of nighttime trips. When we create our dashboards, we can explore this trend more interactively looking at if it changes by year, season etc.


### **6. Fare and Tip Correlation:** Is there a relationship between fare amount and tip amount?

Lets look at the correlation between the trip cost and tip. I am expecting to see some sort of positive correlation
~~~~sql
SELECT
    CORR(fare_amount, tip_amount) AS fare_tip_correlation, CORR(trip_distance, tip_amount) AS trip_tip_correlation
FROM
    taxi_data;
~~~~

| fare_tip_correlation | trip_tip_correlation |
|----------------------|----------------------|
| 0.2716772276640412   | 0.2785495670793727   |


As expected we do see a positive correlation of 26.8% indicating that while it is not incredible strength to it, there is a tendency for trips with higher fares to have slightly higher tip amounts. I have also looked at the correlation between trip distance and tip amount which is also very similar in correlation as expected. Now correlation does not imply causation and there could be various other factors influencing this such as financial situation of the passenger or quality of service both things we have no information on.  However, I think it is a safe assumption to say that there is a general trend to tip more based on the trip distance.



### **7. Trip Type Over the Year: :** Is there any noticeable differences between trip types over the years?

So in this data set we have two types of trips indicated by 1 which is a street hail and 2 which corresponds to a dispatch, I want to write a query to see the percentage of each type has changed over time.

~~~~sql
WITH CTE AS (SELECT EXTRACT (YEAR FROM pickup_date) AS year , CASE WHEN trip_type = 1 THEN 1.0 ELSE 0.0 END AS streethails ,

CASE WHEN trip_type = 2 THEN 1.0 ELSE 0.0 END AS dispatchs

FROM taxi_data)

SELECT year, ROUND((SUM(streethails)/COUNT(*)) * 100,2) as streethail_share, ROUND((SUM(dispatchs)/COUNT(*)) * 100.0,2) AS dispatch_share FROM CTE

GROUP BY YEAR

ORDER BY YEAR ASC
~~~~

| year | streethail_share | dispatch_share |
|------|------------------|----------------|
| 2017 | 98.27            | 1.73           |
| 2018 | 97.30            | 2.70           |
| 2019 | 95.72            | 4.28           |
| 2020 | 97.69            | 2.31           |


Here we can see there really isn’t any noticeable trend over time, given we are just using 4 years of data it would be difficult in general to develop a trend considering that. 


### **8. Longest vs Shortest Trips:** Looking for the longest and shortest trips in the dataset?

~~~~sql
Below is a query returning the data for the longest trip in minutes

WITH CTE AS (SELECT *, ROUND(EXTRACT(EPOCH FROM (dropoff_date_time - pickup_date_time))/60,1) as trip_distance_mins FROM taxi_data)

SELECT * FROM CTE

WHERE trip_distance_mins = (SELECT MAX(trip_distance_mins) FROM CTE) 
~~~~

| vendor_id | pickup_date_time    | dropoff_date_time   | store_and_fwd_flag | ratecode_id | pu_location_id | do_location_id | passenger_count | trip_distance | fare_amount | extra | mta_tax | tip_amount | tolls_amount | improvement_surcharge | total_amount | payment_type | trip_type | congestion_surcharge | pickup_date | pickup_time | dropoff_date | dropoff_time | trip_distance_mins |
|-----------|---------------------|---------------------|--------------------|-------------|----------------|----------------|-----------------|---------------|-------------|-------|---------|------------|--------------|-----------------------|--------------|--------------|-----------|----------------------|-------------|-------------|--------------|--------------|--------------------|
| 2         | 2019-07-30 13:36:32 | 2019-08-05 08:59:08 | N                  | 1           | 193            | 193            | 2               | 0.68          | 1464.00     | 0.80  | 0.50    | 0.00       | 0.00         | 0.00                  | 1465.30      | 2            | 1         | 0.00                 | 2019-07-30  | 13:36:32    | 2019-08-05   | 08:59:08     | 8362.6             |

Here we see the longest trip ever between 2017-2020 is 5 days, now this is incredibly unrealistic but it could be possible as a challenge sort of thing or it could be a system error, there is no way of knowing for sure.

Next is a query finding the shortest trip
~~~~sql
WITH CTE AS (SELECT *, (EXTRACT(EPOCH FROM (dropoff_date_time - pickup_date_time))/60) as trip_distance_mins FROM taxi_data)

SELECT * FROM CTE

WHERE trip_distance_mins = (SELECT MIN(trip_distance_mins) FROM CTE)  

LIMIT 10
~~~~

| vendor_id | pickup_date_time    | dropoff_date_time   | store_and_fwd_flag | ratecode_id | pu_location_id | do_location_id | passenger_count | trip_distance | fare_amount | extra | mta_tax | tip_amount | tolls_amount | improvement_surcharge | total_amount | payment_type | trip_type | congestion_surcharge | pickup_date | pickup_time | dropoff_date | dropoff_time | trip_distance_mins     |
|-----------|---------------------|---------------------|--------------------|-------------|----------------|----------------|-----------------|---------------|-------------|-------|---------|------------|--------------|-----------------------|--------------|--------------|-----------|----------------------|-------------|-------------|--------------|--------------|------------------------|
| 1         | 2018-03-23 00:54:30 | 2018-03-23 00:54:31 | N                  | 1           | 97             | 97             | 1               | 3.20          | 0.00        | 0.00  | 0.00    | 0.00       | 0.00         | 0.00                  | 0.00         | 1            | 1         | 0.00                 | 2018-03-23  | 00:54:30    | 2018-03-23   | 00:54:31     | 0.01666666666666666667 |
| 1         | 2017-01-27 15:40:13 | 2017-01-27 15:40:14 | N                  | 1           | 108            | 108            | 1               | 0.00          | 2.50        | 0.00  | 0.50    | 0.00       | 0.00         | 0.30                  | 3.30         | 2            | 1         | 0.00                 | 2017-01-27  | 15:40:13    | 2017-01-27   | 15:40:14     | 0.01666666666666666667 |
| 2         | 2017-01-02 09:41:39 | 2017-01-02 09:41:40 | N                  | 1           | 25             | 25             | 1               | 0.00          | 2.50        | 0.00  | 0.50    | 0.00       | 0.00         | 0.30                  | 3.30         | 2            | 1         | 0.00                 | 2017-01-02  | 09:41:39    | 2017-01-02   | 09:41:40     | 0.01666666666666666667 |
| 2         | 2017-01-10 13:43:49 | 2017-01-10 13:43:50 | N                  | 1           | 41             | 41             | 1               | 0.00          | 2.50        | 0.00  | 0.50    | 0.00       | 0.00         | 0.30                  | 3.30         | 2            | 1         | 0.00                 | 2017-01-10  | 13:43:49    | 2017-01-10   | 13:43:50     | 0.01666666666666666667 |
| 2         | 2017-01-14 14:25:23 | 2017-01-14 14:25:24 | N                  | 1           | 169            | 169            | 1               | 0.00          | 2.50        | 0.00  | 0.50    | 0.00       | 0.00         | 0.30                  | 3.30         | 2            | 1         | 0.00                 | 2017-01-14  | 14:25:23    | 2017-01-14   | 14:25:24     | 0.01666666666666666667 |
| 2         | 2017-01-15 17:26:54 | 2017-01-15 17:26:55 | N                  | 1           | 60             | 167            | 1               | 0.00          | 2.50        | 0.00  | 0.50    | 0.00       | 0.00         | 0.30                  | 3.30         | 2            | 1         | 0.00                 | 2017-01-15  | 17:26:54    | 2017-01-15   | 17:26:55     | 0.01666666666666666667 |
| 2         | 2017-01-25 14:35:09 | 2017-01-25 14:35:10 | N                  | 1           | 42             | 42             | 1               | 0.00          | 2.50        | 0.00  | 0.50    | 0.00       | 0.00         | 0.30                  | 3.30         | 2            | 1         | 0.00                 | 2017-01-25  | 14:35:09    | 2017-01-25   | 14:35:10     | 0.01666666666666666667 |
| 2         | 2017-01-26 00:21:39 | 2017-01-26 00:21:40 | N                  | 1           | 83             | 83             | 1               | 0.36          | 3.00        | 0.50  | 0.50    | 0.00       | 0.00         | 0.30                  | 4.30         | 2            | 1         | 0.00                 | 2017-01-26  | 00:21:39    | 2017-01-26   | 00:21:40     | 0.01666666666666666667 |
| 2         | 2017-01-06 21:35:32 | 2017-01-06 21:35:33 | N                  | 1           | 193            | 193            | 1               | 0.00          | 2.50        | 0.50  | 0.50    | 0.00       | 0.00         | 0.30                  | 3.80         | 2            | 1         | 0.00                 | 2017-01-06  | 21:35:32    | 2017-01-06   | 21:35:33     | 0.01666666666666666667 |
| 2         | 2017-01-26 21:29:57 | 2017-01-26 21:29:58 | N                  | 1           | 193            | 193            | 1               | 0.00          | 2.50        | 0.50  | 0.50    | 0.00       | 0.00         | 0.30                  | 3.80         | 2            | 1         | 0.00                 | 2017-01-26  | 21:29:57    | 2017-01-26   | 21:29:58     | 0.01666666666666666667 |


Here we can see the shortest trip is one second, this could easily be a system error or maybe it could be someone immediately cancelling once they got in the cab, I am unsure what threshold for time I should use to remove entries like this so I will leave as is. We have a total amount of 2486 entries like this, so it really is so insignificant compared to the 27 trips we have recorded. Something like this in a real-world scenario should be communicated with other analysts and team members to come up with an appropriate cutoff threshold.


### **9. Trip Duration Analysis:** What's the average duration of trips for different passenger counts, is there any trends?
Below is a query returning the average trip distance for each passenger count.
~~~~sql
SELECT passenger_count, ROUND(AVG(trip_distance)*1.60934,2) as average_trip_distance FROM taxi_data

GROUP BY passenger_count

ORDER BY passenger_count ASC
~~~~

| passenger_count | average_trip_distance |
|-----------------|-----------------------|
| 1               | 4.68                  |
| 2               | 4.81                  |
| 3               | 4.90                  |
| 4               | 4.76                  |
| 5               | 4.71                  |
| 6               | 4.42                  |

Here we can see no noticeable trend for average distance depending on passenger count.

### **10. Payment Trends Over Time:** How have the proportions of different payment types (credit card, cash, etc.) changed over the years?

Here is a query showing the payment trends over time, I am expecting that credit card and cash to continually be the most popular as the other payment types are quite unordinary.

~~~~sql
WITH CTE AS (SELECT EXTRACT (YEAR FROM dropoff_date) AS year, CASE WHEN payment_type = 1 then 1.0 else 0.0 end as credit_count, 
CASE WHEN payment_type = 2 then 1.0 else 0.0 end as cash_count,
CASE WHEN payment_type = 3 then 1.0 else 0.0 end as no_charge_count,
CASE WHEN payment_type = 4 then 1.0 else 0.0 end as dispute_count,
CASE WHEN payment_type = 5 then 1.0 else 0.0 end as unknown_count 
FROM taxi_data)


SELECT year, ROUND((SUM(credit_count)/COUNT(*)) * 100,2) as credit_share, ROUND((SUM(cash_count)/COUNT(*)) * 100,2) as cash_share,
ROUND((SUM(no_charge_count)/COUNT(*)) * 100,2) as no_charge_share, ROUND((SUM(dispute_count)/COUNT(*)) * 100,2) as dispute_share,
ROUND((SUM(unknown_count)/COUNT(*)) * 100,2) as unknown_share FROM CTE


GROUP BY year

ORDER BY year ASC
~~~~

| year | credit_share | cash_share | no_charge_share | dispute_share | unknown_share |
|------|--------------|------------|-----------------|---------------|---------------|
| 2017 | 50.50        | 49.00      | 0.31            | 0.19          | 0.00          |
| 2018 | 57.02        | 42.56      | 0.27            | 0.14          | 0.00          |
| 2019 | 57.19        | 42.40      | 0.28            | 0.13          | 0.00          |
| 2020 | 55.19        | 44.34      | 0.36            | 0.10          | 0.00          |
| 2021 | 55.56        | 44.44      | 0.00            | 0.00          | 0.00          |

Here we see a sharp increase from 2017 to 2018 for credit card and a corresponding drop for cash and then 2019 and 2020 are relatively close for credit cards share, so credit card has increased drastically from 2017 and evening out about +- a couple percent for the remaining years. 2021 also appears in here and I decided to keep it in because the pickup date field is still contained in 2020 but the final drop off date was during the first day of 2021.

~~~~sql
SELECT * FROM taxi_data

WHERE EXTRACT(YEAR FROM dropoff_date_time) = 2021
~~~~

| vendor_id | pickup_date_time    | dropoff_date_time   | store_and_fwd_flag | ratecode_id | pu_location_id | do_location_id | passenger_count | trip_distance | fare_amount | extra | mta_tax | tip_amount | tolls_amount | improvement_surcharge | total_amount | payment_type | trip_type | congestion_surcharge | pickup_date | pickup_time | dropoff_date | dropoff_time |
|-----------|---------------------|---------------------|--------------------|-------------|----------------|----------------|-----------------|---------------|-------------|-------|---------|------------|--------------|-----------------------|--------------|--------------|-----------|----------------------|-------------|-------------|--------------|--------------|
| 2         | 2020-12-31 23:08:58 | 2021-01-01 00:09:43 | N                  | 1           | 244            | 89             | 3               | 22.10         | 66.50       | 0.50  | 0.50    | 0.00       | 0.00         | 0.30                  | 70.55        | 2            | 1         | 2.75                 | 2020-12-31  | 23:08:58    | 2021-01-01   | 00:09:43     |
| 2         | 2020-12-31 23:53:51 | 2021-01-01 00:18:10 | N                  | 1           | 75             | 186            | 1               | 4.40          | 19.00       | 0.50  | 0.50    | 0.00       | 0.00         | 0.30                  | 23.05        | 2            | 1         | 2.75                 | 2020-12-31  | 23:53:51    | 2021-01-01   | 00:18:10     |
| 2         | 2020-12-31 16:41:12 | 2021-01-01 16:00:07 | N                  | 1           | 75             | 224            | 1               | 5.60          | 19.50       | 1.00  | 0.50    | 0.00       | 0.00         | 0.30                  | 21.30        | 2            | 1         | 0.00                 | 2020-12-31  | 16:41:12    | 2021-01-01   | 16:00:07     |
| 2         | 2020-12-31 23:59:53 | 2021-01-01 00:03:26 | N                  | 1           | 75             | 74             | 1               | 1.34          | 5.50        | 0.50  | 0.50    | 1.36       | 0.00         | 0.30                  | 8.16         | 1            | 1         | 0.00                 | 2020-12-31  | 23:59:53    | 2021-01-01   | 00:03:26     |
| 2         | 2020-12-31 08:48:25 | 2021-01-01 08:35:08 | N                  | 1           | 35             | 35             | 1               | 9.46          | 35.50       | 0.00  | 0.50    | 0.00       | 0.00         | 0.30                  | 36.30        | 2            | 1         | 0.00                 | 2020-12-31  | 08:48:25    | 2021-01-01   | 08:35:08     |
| 2         | 2020-12-31 23:41:27 | 2021-01-01 00:03:44 | N                  | 1           | 41             | 68             | 1               | 4.91          | 18.50       | 0.50  | 0.50    | 4.51       | 0.00         | 0.30                  | 27.06        | 1            | 1         | 2.75                 | 2020-12-31  | 23:41:27    | 2021-01-01   | 00:03:44     |
| 1         | 2020-12-31 23:51:58 | 2021-01-01 00:11:07 | N                  | 1           | 75             | 169            | 1               | 5.50          | 18.50       | 0.50  | 0.50    | 2.00       | 0.00         | 0.30                  | 21.80        | 1            | 1         | 0.00                 | 2020-12-31  | 23:51:58    | 2021-01-01   | 00:11:07     |
| 2         | 2020-12-31 23:54:15 | 2021-01-01 00:11:17 | N                  | 1           | 75             | 167            | 1               | 3.74          | 15.00       | 0.50  | 0.50    | 1.00       | 0.00         | 0.30                  | 17.30        | 1            | 1         | 0.00                 | 2020-12-31  | 23:54:15    | 2021-01-01   | 00:11:17     |
| 2         | 2020-12-31 14:41:30 | 2021-01-01 13:49:19 | N                  | 1           | 74             | 42             | 1               | 1.45          | 7.50        | 0.00  | 0.50    | 0.00       | 0.00         | 0.30                  | 8.30         | 1            | 1         | 0.00                 | 2020-12-31  | 14:41:30    | 2021-01-01   | 13:49:19     |

# Conclusion

In conclusion this was a cool project getting insight into 27 million plus real taxi trips in NYC from 2017-2020, many of my justifications for cleaning are my own opinion and other people may have another idea of how to approach cleaning and dropping particular values. This is something that given a real-world situation I would take extra caution ensuring that I ask multiple people for their inputs, which is something I cannot do particular for this as I am the sole analyst doing this project.

This case study demonstrating many fundamental SQL skills including data cleaning, joins, CTES, agg
