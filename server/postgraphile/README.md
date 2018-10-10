# References

## Time-series databases
* [Top 10 open source time-series databases, as of August 2016](https://www.outlyer.com/blog/top10-open-source-time-series-databases/)
* [Building scalable time-series database on PostgreSQL](https://blog.timescale.com/when-boring-is-awesome-building-a-scalable-time-series-database-on-postgresql-2900ea453ee2)
* [Getting started with TimescaleDB](https://docs.timescale.com/v0.12/getting-started)

## GraphQL
* [Introduction to PostGraphile (GraphQL on top of PostgreSQL)](http://www.graphile.org/postgraphile/introduction/)

# Installation

## TimescaleDB extension

### MacOS
* Add the Homebrew tap for TimescaleDB:
```bash
$ brew tap timescale/tap
```

* Install TimescaleDB:
```bash
$ brew install timescaledb
$ /usr/local/bin/timescaledb_move.sh
```

* Add TimescaleDB library in PostgreSQL configuration.
  In the ``/usr/local/var/postgres/postgresql.conf`` file,
  add ``timescaledb`` to the (comma-separated) list of libraries.
  For instance:
```conf
shared_preload_libraries = 'timescaledb'
```

* Re-start PostgreSQL:
```bash
$ brew services restart postgresql
```

* Create a ``postgres`` superuser:
```bash
$ createuser postgres -s
```

* Check that everything went fine:
```bash
$ brew services list | grep postgre
```

### Create a Timescale-enabled database
* Start a SQL session:
```bash
$ psql -U postgres -h localhost
```

* Create a ``tutorial`` database, connect to it
```sql
postgres=# CREATE database tutorial;
CREATE DATABASE
postgres=# \c tutorial
You are now connected to database "tutorial" as user "postgres".
tutorial=# CREATE EXTENSION IF NOT EXISTS timescaledb CASCADE;
WARNING:  
WELCOME TO
 _____ _                               _     ____________  
|_   _(_)                             | |    |  _  \ ___ \ 
  | |  _ _ __ ___   ___  ___  ___ __ _| | ___| | | | |_/ / 
  | | | |  _ ` _ \ / _ \/ __|/ __/ _` | |/ _ \ | | | ___ \ 
  | | | | | | | | |  __/\__ \ (_| (_| | |  __/ |/ /| |_/ /
  |_| |_|_| |_| |_|\___||___/\___\__,_|_|\___|___/ \____/
               Running version 0.12.1
For more information on TimescaleDB, please visit the following links:

 1. Getting started: https://docs.timescale.com/getting-started
 2. API reference documentation: https://docs.timescale.com/api
 3. How TimescaleDB is designed: https://docs.timescale.com/introduction/architecture

Note: TimescaleDB collects anonymous reports to better understand and assist our users.
For more information and how to disable, please see our docs https://docs.timescaledb.com/using-timescaledb/telemetry.

CREATE EXTENSION
tutorial=# \q
```

* Start a TimescaleDB session (on the ``tutorial`` database):
```bash
$ psql -U postgres -h localhost -d tutorial
```

### Create a (Hyper)table
* Reference: https://docs.timescale.com/v0.12/getting-started/creating-hypertables

* Create a regular table and transform it:
```sql
tutorial=# DROP TABLE if exists conditions;

tutorial=# CREATE TABLE conditions (
  id             INTEGER           NULL,
  time           TIMESTAMPTZ       NOT NULL,
  temperature    DOUBLE PRECISION  NULL,
  humidity       DOUBLE PRECISION  NULL,
  light          DOUBLE PRECISION  NULL,
  co2            DOUBLE PRECISION  NULL,
  humidity_ratio DOUBLE PRECISION  NULL,
  occupancy      BOOLEAN           NULL
);
CREATE TABLE

tutorial=# SELECT create_hypertable('conditions', 'time');
 create_hypertable 
-------------------
 
(1 row)
```

* Insert records:
```sql
tutorial=# INSERT INTO conditions(time, location, temperature, humidity) VALUES (NOW(), 'office', 70.0, 50.0);
tutorial=# INSERT INTO conditions(time, location, temperature, humidity) VALUES (NOW(), 'office', 100.0, 20.0);
tutorial=# INSERT INTO conditions(time, location, temperature, humidity) VALUES (NOW(), 'office', 80.0, 30.0);
tutorial=# INSERT INTO conditions(time, location, temperature, humidity) VALUES (NOW(), 'office', 90.0, 40.0);
```

* Get records:
```sql
tutorial=# SELECT * FROM conditions ORDER BY time DESC LIMIT 100;
             time              | location | temperature | humidity 
-------------------------------+----------+-------------+----------
 2018-10-03 20:58:42.557148+02 | office   |          90 |       20
 2018-10-03 20:58:39.753109+02 | office   |          90 |       40
 ... 
 2018-10-03 20:55:26.631314+02 | office   |          70 |       50
(12 rows)
```

### Load the occupancy data set into the database
"date","Temperature","Humidity","Light","CO2","HumidityRatio","Occupancy"
"1","2015-02-04 17:51:00",23.18,27.272,426,721.25,0.00479298817650529,1

* [Reference for the ``\copy`` command in PostgreSQL](https://www.postgresql.org/docs/current/static/sql-copy.html)
```bash
$ bzcat ../../data/time-series/datatraining.txt.bz2 | psql -U postgres -h localhost -d tutorial -c "copy conditions(id,time,temperature,humidity,light,co2,humidity_ratio,occupancy) from stdin delimiter ',' csv header;"
COPY 8143
```

## PostGraphile

### Node.js

#### MacOS
```bash
$ brew install node
```

### Postgraphile
```bash
$ npm install -g postgraphile
```

* [Inflection plug-in](https://www.graphile.org/postgraphile/inflection) (optional):
```bash
$ npm install -g postgraphile @graphile-contrib/pg-simplify-inflector
$ postgraphile --append-plugins @graphile-contrib/pg-simplify-inflector
```

# Sample queries

## Launch PostGraphile
```bash
$ postgraphile -c "postgres://postgres@localhost/tutorial" -a -j --watch
```

## Extract temperature and humidity for all the records
* Open a Web browser on the [local GraphiQL just launched application](http://localhost:5000),
  copy and paste the following JSON query, and execute it:
```json
query conditions_first_10 {
  allConditions(first: 10) {
    nodes {
    	time,
    	temperature,
    	humidity,
      light,
      co2,
      humidityRatio,
      occupancy
    }
  }
}
{
  "data": {
    "allConditions": {
      "nodes": [
        {
          "time": "2015-02-04T17:51:00+01:00",
          "temperature": 23.18,
          "humidity": 27.272,
          "light": 426,
          "co2": 721.25,
          "humidityRatio": 0.00479298817650529,
          "occupancy": true
        },
        {
          "time": "2015-02-04T17:51:59+01:00",
          "temperature": 23.15,
          "humidity": 27.2675,
          "light": 429.5,
          "co2": 714,
          "humidityRatio": 0.00478344094931065,
          "occupancy": true
        },
...
        {
          "time": "2015-02-04T18:00:00+01:00",
          "temperature": 23.075,
          "humidity": 27.175,
          "light": 419,
          "co2": 688,
          "humidityRatio": 0.00474535071966655,
          "occupancy": true
        }
      ]
    }
  }
}
```



