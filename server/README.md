# References

## Time-series databases
* [Top 10 open source time-series databases, as of August 2016](https://www.outlyer.com/blog/top10-open-source-time-series-databases/)
* [Building scalable time-series database on PostgreSQL](https://blog.timescale.com/when-boring-is-awesome-building-a-scalable-time-series-database-on-postgresql-2900ea453ee2)
* [Getting started with TimescaleDB](https://docs.timescale.com/v0.12/getting-started)

# Installation
## MacOS
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

## Create a Timescale-enabled database
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

## Create a (Hyper)table
* Reference: https://docs.timescale.com/v0.12/getting-started/creating-hypertables

* Create a regular table and transform it:
```sql

tutorial=# CREATE TABLE conditions (
  time        TIMESTAMPTZ       NOT NULL,
  location    TEXT              NOT NULL,
  temperature DOUBLE PRECISION  NULL,
  humidity    DOUBLE PRECISION  NULL
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


