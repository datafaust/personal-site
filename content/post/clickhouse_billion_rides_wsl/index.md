+++
title = "A Billion Taxi Rides in Clickhouse WSL"
subtitle = "All on a Dell G7:rocket:"

date = 2018-12-07T00:00:00
lastmod = 2018-12-07T00:00:00
draft = false

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ['Fausto Lopez']

tags = ["Academic"]
summary = "Loading a billion taxi rides into clickhouse on WSL as a proof of concept for low resource optimization."

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["deep-learning"]` references 
#   `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
# projects = ["internal-project"]

# Featured image
# To use, add an image named `featured.jpg/png` to your project's folder. 
[image]
  # Caption (optional)
  caption = ""

  # Focal point (optional)
  # Options: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight
  focal_point = ""

  # Show image only in page previews?
  preview_only = false

# Set captions for image gallery.
+++

## Why ClickHouse?

Recently I've been wanting to expand my knowledge of other database types beyond SQL variants and map reduce methods with spark. As a data sceintist at the Taxi & Limousine Commission I follow some of the work that is done by the public using our data and have followed [Mark Litwintschik's blog](https://tech.marksblogg.com/) blog for some time. At work we currently lean on SQL Server and PostgreSQL along with Python and R for most of our analyses but I've been curious about incentivizing change (albeit a resistant IT department as many of you probably have).

In 2017 Mark benchmarked 1.1 billion taxi rides using [Todd Schneider's](https://github.com/toddwschneider/nyc-taxi-data) ETL scripts with our public data. It's been some time but I thought I'd give it a shot myself, particularly since I was intrigued by the performance gains over postgres and even spark on a single node machine.  

I didn't want to spin up a cluster on Amazon to pay at the moment, ot to mention all the storage I would need to buy, so I decided to run this on my Dell G7 laptop which has some impresive specs for a cheap laptop:

![](/img/clickhouse_single_node/specs.png)

Since clickhouse only runs on linux distributions I chose to run it on Ubuntu 18.04 on WSL (Windows Subsystem Linux). The reason I didn't choose virtualbox was because I wanted to leverage the full power on my computer, and later test Windows HouseOps functionality to access the server locally, a nifty setup to convince the higher ups to use this. 

##Getting WSL up and Running

Installing wsl in windows is simple, with [documentation here](https://docs.microsoft.com/en-us/windows/wsl/install-win10). Simply activate via powershell and then install through the windows store (choice 1 from the 3 choices listed in the documentation).

Once Ubuntu is installed, you'll want to open up ubuntu through cortana and begin standard linux setup, adding in your password and user information. You'll want to run the usual update and upgrade values:

`sudo apt-get update && sudo apt-get upgrade`

Since I was just testing out the system and wsl is behind windows firewall I didn't setup any firewalls or security. 

##PostgreSQL

I followed Mark's blog on coding the data through [postgres](https://tech.marksblogg.com/billion-nyc-taxi-rides-redshift.html) in order to produce the gz files we will load into clickhouse. You can follow the instructions there but here is a short summary of what to do. You'll want to install postgres client and server. In ubuntu, make a directory for the taxi data and clone Todd's repo (note I chose to place my files in windows in order to play around with accessing files across platforms; this means you will need to access the windows files directories):

`sudo apt-get install postgresql`
`mkdir /mnt/c/Users/0000/Desktop/Documents`
`cd /mnt/c/Users/0000/Desktop/Documents/nyc-taxi-data`
`git clone https://github.com/toddwschneider/nyc-taxi-data.git`
`cd /mnt/c/Users/0000/Desktop/Documents/nyc-taxi-data`

you'll download the data automatically, but remember that Todd's script pulls the urls of ALL the trip records. If you just want to test first with only a year or so of data I reccomend opening the url script with something like:

`sudo nano /mnt/c/Users/0000/Desktop/Documents/nyc-taxi-data/setup_files/raw_data_urls.txt`

and deleting as many files as you wish, so you can run:

`./download_raw_data.sh && ./remove_bad_rows.sh`

Once you've downloaded the data, you can run the initialize and import scripts:

`./import_trip_data.sh` 
`./import_fhv_trip_data.sh`

I chose to do this in chunks as I did not have the hard drive space in place. This is definately the longest part of the process. Once it's done, you will now have a postgresql database with all the data loaded in which you can query. This means we can pull the data out and compress it into .gz files which will be loaded into clickhouse. You'll want to:

1. create the directory: `mkdir -p /mnt/c/Users/0000/Desktop/Documents/nyc-taxi-data/trips`
2. add permissions to write to the directory: `sudo chown -R postgres:postgres \`
    `/mnt/c/Users/0000/Desktop/Documents/nyc-taxi-data/trips`
3. enter postgres: `psql nyc-taxi-data`

And then run the script to pull the data:

```
COPY (
    SELECT trips.id,
           trips.vendor_id,
           trips.pickup_datetime,
           trips.dropoff_datetime,
           trips.store_and_fwd_flag,
           trips.rate_code_id,
           trips.pickup_longitude,
           trips.pickup_latitude,
           trips.dropoff_longitude,
           trips.dropoff_latitude,
           trips.passenger_count,
           trips.trip_distance,
           trips.fare_amount,
           trips.extra,
           trips.mta_tax,
           trips.tip_amount,
           trips.tolls_amount,
           trips.ehail_fee,
           trips.improvement_surcharge,
           trips.total_amount,
           trips.payment_type,
           trips.trip_type,
           trips.pickup_location_id,
           trips.dropoff_location_id,

           cab_types.type cab_type,

           weather.precipitation rain,
           weather.snow_depth,
           weather.snowfall,
           weather.max_temperature max_temp,
           weather.min_temperature min_temp,
           weather.average_wind_speed wind,

           pick_up.gid pickup_nyct2010_gid,
           pick_up.ctlabel pickup_ctlabel,
           pick_up.borocode pickup_borocode,
           pick_up.boroname pickup_boroname,
           pick_up.ct2010 pickup_ct2010,
           pick_up.boroct2010 pickup_boroct2010,
           pick_up.cdeligibil pickup_cdeligibil,
           pick_up.ntacode pickup_ntacode,
           pick_up.ntaname pickup_ntaname,
           pick_up.puma pickup_puma,

           drop_off.gid dropoff_nyct2010_gid,
           drop_off.ctlabel dropoff_ctlabel,
           drop_off.borocode dropoff_borocode,
           drop_off.boroname dropoff_boroname,
           drop_off.ct2010 dropoff_ct2010,
           drop_off.boroct2010 dropoff_boroct2010,
           drop_off.cdeligibil dropoff_cdeligibil,
           drop_off.ntacode dropoff_ntacode,
           drop_off.ntaname dropoff_ntaname,
           drop_off.puma dropoff_puma
    FROM trips
    LEFT JOIN cab_types
        ON trips.cab_type_id = cab_types.id
    LEFT JOIN central_park_weather_observations weather
        ON weather.date = trips.pickup_datetime::date
    LEFT JOIN nyct2010 pick_up
        ON pick_up.gid = trips.pickup_nyct2010_gid
    LEFT JOIN nyct2010 drop_off
        ON drop_off.gid = trips.dropoff_nyct2010_gid
) TO PROGRAM
    'split -l 20000000 --filter="gzip > /mnt/c/Users/0000/Desktop/db/latest/nyc-taxi-data/trips/trips_\$FILE.csv.gz"'
    WITH CSV;
	
```

Once you finish downloading the files you are read to load them into clickhouse. The beauty of these files is that you can use them again and again to explore some of the other databases that Mark benchmarks or otherwise explore options you may want to try.

##Setting up clickhouse in wsl

This was certainly the trickiest part of this porcess. Clickhouse [documentation](https://clickhouse.yandex/docs/en/getting_started/) asks for the following code segments:

```
deb http://repo.yandex.ru/clickhouse/deb/stable/ main/
sudo apt-get install dirmngr    # optional
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv E0C56BD4    # optional
sudo apt-get update
sudo apt-get install clickhouse-client clickhouse-server
```

If you run these in wsl you will receive an error that says dbgmanager is not installed. WSL to my knowledge has not solved CURL issues through dbmanager so changing keys from sources proves difficult. There are a few options noted in serveral forums that you can see [here](https://github.com/Microsoft/WSL/issues/3286), but in my case I decided to install clickhouse manually. Simply go to the [repository](http://repo.yandex.ru/clickhouse/deb/stable/main/) and install matching versions, of the commons, server and client:

```{r eval = F}
clickhouse-common-static_19.1.6_amd64.deb
clickhouse-server_19.1.6_all.deb
clickhouse-client_19.1.6_all.deb

```

In my case they hit the downloads folder which you can reference and install:

`cd /mnt/c/Users/0000/Desktop/Downloads`
`sudo dpkg -i /path/to/deb/file`

Note you will do this for al three files. Follow the prompts and you should be able to install the software without a hitch. Once everything is installed, start the service:

`sudo service clickhouse-server start`

Start the client:

`clickhouse client`

You are now in the equivalent of the psql database in clickhouse.In here you can run all your database queries, you can create a database and begin making tables. You will want to create the trips table in order to populate it with the data from the .gz files. You can run the following code:

```
CREATE TABLE trips (
    id                      UInt32,
    vendor_id               String,

    pickup_datetime         DateTime,
    dropoff_datetime        Nullable(DateTime),

    store_and_fwd_flag      Nullable(FixedString(1)),
    rate_code_id            Nullable(UInt8),
    pickup_longitude        Nullable(Float64),
    pickup_latitude         Nullable(Float64),
    dropoff_longitude       Nullable(Float64),
    dropoff_latitude        Nullable(Float64),
    passenger_count         Nullable(UInt8),
    trip_distance           Nullable(Float64),
    fare_amount             Nullable(Float32),
    extra                   Nullable(Float32),
    mta_tax                 Nullable(Float32),
    tip_amount              Nullable(Float32),
    tolls_amount            Nullable(Float32),
    ehail_fee               Nullable(Float32),
    improvement_surcharge   Nullable(Float32),
    total_amount            Nullable(Float32),
    payment_type            Nullable(String),
    trip_type               Nullable(UInt8),
    pickup_location_id      Nullable(UInt8),
    dropoff_location_id     Nullable(UInt8),

    cab_type                Nullable(String),

    rain		            Nullable(Float32),
    snow_depth              Nullable(Float32),
    snowfall                Nullable(Float32),
    max_temperature         Nullable(Float32),
    min_temperature         Nullable(Float32),
    wind			        Nullable(Float32),

    pickup_nyct2010_gid     Nullable(Int8),
    pickup_ctlabel          Nullable(String),
    pickup_borocode         Nullable(Int8),
    pickup_boroname         Nullable(String),
    pickup_ct2010           Nullable(String),
    pickup_boroct2010       Nullable(String),
    pickup_cdeligibil       Nullable(FixedString(1)),
    pickup_ntacode          Nullable(String),
    pickup_ntaname          Nullable(String),
    pickup_puma             Nullable(String),

    dropoff_nyct2010_gid    Nullable(UInt8),
    dropoff_ctlabel         Nullable(String),
    dropoff_borocode        Nullable(UInt8),
    dropoff_boroname        Nullable(String),
    dropoff_ct2010          Nullable(String),
    dropoff_boroct2010      Nullable(String),
    dropoff_cdeligibil      Nullable(String),
    dropoff_ntacode         Nullable(String),
    dropoff_ntaname         Nullable(String),
    dropoff_puma            Nullable(String)
) ENGINE = Log;
```

and you should receive the following message:

![](/img/clickhouse_single_node/create_table.png)

You should be all set now to load the trips.

##Loading the data into clickhouse

Loading the data into clickhouse requires editing some of the fields in the trip data. Mark writes:

-The dataset itself uses commas for delimiting fields. None of the contents of the data contains any commas themselves so there is no quotations used to aid escaping data. NULL values are defined by the simple absence of any content between the comma delimiters. Normally this isn't an issue but with ClickHouse empty fields won't be treated as NULLs in order to avoid ambiguity with empty strings. For this reason I need to pipe the data through a transformation script that will replace all empty values with \N.

So in order to fix this we run his script. You will want to make sure that you install python and create the script:

`sudo apt-get install python`
`sudo nano trans.py`

and then paste in the script text:

```
import sys


for line in sys.stdin:
    print ','.join([item if len(item.strip()) else '\N'
                    for item in line.strip().split(',')])
```

Now you can run the upload script:

```{r eval = F}
time (for filename in /mnt/c/Users/0000/Desktop/Documents/nyc-taxi-data/trips/trips_x*.csv.gz; do
            gunzip -c $filename | \
                python trans.py | \
                clickhouse-client \
                    --query="INSERT INTO trips FORMAT CSV"
        done)
```

You should now see clickhouse loading and processing data. This will vary on the amount of data but shouldn't take more than a few hours tops.

##Query and Test 
You can take this a step further, creating a mergetree table and tuning up. But you will want to run a few tests first. You can go head and access the data with some basic queries:

```
----cab type
SELECT cab_type, count(*)
FROM trips
GROUP BY cab_type

----puloc double group by
select 
pickup_location_id,
cab_type,
count(pickup_datetime)
FROM
trips
group by
pickup_location_id,
cab_type
```

You should see these queries run in a few seconds depending on how much data you loaded.

##Conclusions and Next Steps
Ultimately this is a very powerful software, particularly for low-resource departments with big data usage. This setup is certainly not the most optimal, but for quick access to large data for the average person, clickhouse seems to be a very powerful tool with great potential. A better setup would leverage pure linux distributions, but this can be a great option for organizations which for security reasons may not be able to run linux outside of a windows environment.In the next step I will try and run Tabix, which is a third party gui meant to allow for access of clickhouse data as well as test R and Python access to clickhouse via wsl. Stay tuned!


