# Home Assistant - Maintenance

## Table of content

- Regular maintenance:
  - [Check updates for Home Assistant](https://github.com/slittorin/home-assistant-maintenance#check-updates-for-home-assistant)
  - [Check updates for server1](https://github.com/slittorin/home-assistant-maintenance#check-updates-for-server1)
  - [Check updates for InfluxDB](https://github.com/slittorin/home-assistant-maintenance#check-updates-for-influxdb)
  - [Check updates for Grafana](https://github.com/slittorin/home-assistant-maintenance#check-updates-for-grafana)
  - [Remove obsolete entities](https://github.com/slittorin/home-assistant-maintenance#remove-obsolete-or-faulty-entities)
  - [Add domain sensors](https://github.com/slittorin/home-assistant-maintenance#add-domain-sensors)
  - [Exclude sensors for InfluxDB integration](https://github.com/slittorin/home-assistant-maintenance#exclude-sensors-for-influxdb-integration)
- Errors, problems and challenges:
  - [Incorrect SMA Energy data](https://github.com/slittorin/home-assistant-maintenance#incorrect-sma-energy-data)
  - [Incorrect Balboa Spa data](https://github.com/slittorin/home-assistant-maintenance#incorrect-balboa-spa-data)

## Regular maintenance

### Check updates for Home Assistant

Check regularly if there is any update for Home Assistant: Core, Operation System or Supervisor.\
Install through the UI.\
Note that you are encoruaged to take backup prior to any upgrade.

### Check updates for server1

Check regularly if there is any update for server1 (Raspberry PI).\
Install through normal `apt` commands.

### Check updates for InfluxDB

Check regularly if there is any important update that is required for InfluxDB docker-container.\
Note that you are encoruaged to take backup prior to any upgrade.

Check [Setup instructions](https://github.com/slittorin/home-assistant-setup#installation-for-influxdb).

### Check updates for Grafana

Check regularly if there is any important update that is required for Grafana docker-container.\
Note that you are encoruaged to take backup prior to any upgrade.

Check [Setup instructions](https://github.com/slittorin/home-assistant-setup#installation-for-grafana).

### Remove obsolete or faulty entities

From time to time, specifically when you are testing and adding capabilities, there will be obsolete entities/sensors that can be removed.

Perform the following:
- In Home Assistant through `Developer Tools` -> `Statistics` tab:
- For entities marked with `Fix issue`.
- Isolate first if the entity can be removed:
  - For instance test related sensors, or entities created during dev/test.
  - Click on the row (not `Fix issue`) and `Remove entity'.
- For `The unit of this entity changed from`:
  - If you want to correct the unit in the database, press `Fix issue` and `Update the historical statistics values`.
- For `There is no state availabile for this entity`:
  - If you want to remove this entity, press `Fix issue` and `Remove`.
- For `The unit of this entity changed from`: 
  - If you want to remove this entity, click on the row (not `Fix issue`) and `Remove entity'.
- For `The unit of this entity doesn't atch a unit of the device class`:
  - Correct the `unit_of_measurement` in the yaml-file and reload the sensors (or restart HA).

### Add domain sensors

With they way we are tracking data, we need add sensors when we add integrations/add-ons/devices to our HA system.

1. Keep track if there are new domains that are not tracked.
   - Go to Dashboard Home Assistant.
   - Check the listed domains towards those documented in [Domains and Entities configuration](https://github.com/slittorin/home-assistant-configuration#package---home-assistant-system---domains-and-entities).
2. Where required, add domains and sensors.
   - Update also [Visualization for Number of domains and entities](https://github.com/slittorin/home-assistant-visualization#number-of-domains-and-entities).

### Exclude sensors for InfluxDB integration

See first [Governing principles](https://github.com/slittorin/home-assistant-setup#governing-principles) on how Historical data and Database retention is setup.

There are some integrations that will generate a huge amount of states, for instance the SMA Inverter and Home Manager integration will refresh each 5 second and give new state/events.

And as we normally cannot change the poll/scan-interval through yaml-configuration for integrations, we only have the option to either disable `Enable polling for updates` for the integration (and make manual trigger for service `homeassistant.update_entity`), or to exclude sensor/states to be written.

Integrations with swift state changes will fill up the states and events table in the Recorder-database, but that is ok with our setup as we run MariaDB and have lots of space. Also the data will roll-over when `purge_keep_days` hits the data.\
However, states will still be written to the InfluxDB database, and therefore over time cause the database to be substantially large.

Therefore we need from time to time to analyze the number of events/states in the database, to get a sense on what type of data that is written and if it is necessary to exclude some sensors from writing data to InfluxDB.

Note here that at this point we do not care about the data volume in the `statistics` table as this gets data written once per hour (`statistics_short_term` gets written every 5 minutes, but is purged according to Recorder setting).

How I did the analysis for my setup:
1. Run the following sql-command in for instance MySQL Workbench, to get the 30 tables with most rows, in percentage of rows.
   ```sql
   select count(*) into @rows from homeassistant.states;
   select entity_id, ((count(distinct state_id)/@rows)*100) as Pct from homeassistant.states group by entity_id order by COUNT(*) desc limit 0,30;
   ```
2. Base on the output, you need to investigate what these sensors provide for value.
  - Such as `sensor.metering_current_l1` (1 through 3) is the ampere every 5 seconds, and accounts for 8% of by states-database (that is roughly 900MB in size for 30 days of data).
    - Take that times 3 for the ampere, and similar for `sensor.metering_active_power_draw_l1` (1 through 3), and that will account for 46% of the states-table, and hence also account for 46% of the data written to InfluxDB (roughly 430MB per month).
    - It is more sense to not write this to the Influx-database, but instead add a statistics-sensor that will keep track of the min, max and mean-values for the last hour (or similar).
      - We either create the sensors ourselves, or allow HA to get the data from the `statistics` table with history graph.
        - Note however that not all data is written to the history tables. See [Governing principles in setup](https://github.com/slittorin/home-assistant-setup#governing-principles).
          - Check with the following (change statistics_id to your full entity id) if there is data in the history tables:
            ```sql
            select * from homeassistant.statistics_meta where statistic_id = 'sensor.metering_current_l1'
            ```
            If it is not, you may want to create the statistics sensors yourself.

## Errors, problems and challenges:

### Incorrect SMA Energy data

The data in Home Assistant for SMA Inverter and Home Manager 2.0 has been wrong.

In this case, there was a peak at 25/1-2022 and 1/2-2022 where produced (consumed) solar was over 2700 kWh early in the morning.

Others have also had this problem, see [here](https://community.home-assistant.io/t/sma-solar-sensor-pv-gen-meter-showing-inconsistent-data/368280) and [here](https://github.com/home-assistant/core/issues/61838).

Note that history graph for `sensor.total_yield` and `sensor.pv_gen_meter` still has a dip to zero in history graph even after correcting the data according to below, possibly related to data not being present in the `states` table (see below) (or not properly deleted/updated).

### This is how I identified and corrected the problem.

#### For Recorder database (MariaDB) i am using MySQL Workbench and excel to correct:

Note that one only has 5 minute window to correct, otherwise `statistics_short_term` table will have new data to correct (and if over each hour, `statistics` table.

Be extra cautious with the sql-update commands, preferably take a backup before updating.\
Use for instance MySQL Workbench to perform sql commands.

1. Identify the different meta-id that you need to look at through `select * from homeassistant.statistics_meta`.
   - You may need to identify several that applies to the error.
   - The problem was identified to occur from roughly 2022-01-24 19:00 to roughly 2022-01-25 06:00.
   - For this problem the following sensor-data was identified to investigate (all have the unit kWh):
     - metadata_id 4: `sensor.total_yield`.
     - metadata_id 5: `sensor.pv_gen_meter`.
     - metadata_id 8: `sensor.metering_total_yield`.
     - metadata_id 9: `sensor.metering_total_absorbed`.
     - metadata_id 17: `sensor.metering_total_yield_compensation`.
   - Sensor data for these, resides in the following statistics-tables:
     - `statistics` for hourly data.
     - `statistics_short_term` for 5 minute data.
2. For the identified meta-ids, run sql command to look at the statistics data:
   - With:
     - `select * from homeassistant.TABLE where metadata_id = METADATA_ID order by created desc;`.
   - For this problem, it occured when `state` was set to 0 (zero).
3. First we need to correct where `state` is 0 (zero) for identified statistics-tables and metadata_ids.
   - Do this by updating `state` column to the last valid value (data is not updating during the period).
4. Then we need to correct `sum` column so it is correctly reflects the increase for the statistics-tables.
   - Do this by updating the `sum` column from the where it is clear that the sum-increase is wrong.
5. Remember to verify that no new values has been written to the tables.
   - If so, they need to be updated.
6. For the `states` table We there was usually no data saved when the error occured (or 'unknown'/'unavailable' was written), but the following sensors needed correction (we corrected these, even though they will be removed when purge-period comes):
   - `sensor.total_yield` - One state_id was wrong.
   - `sensor.pv_gen_meter` - One state_id was wrong.
   - `sensor.metering_total_yield` - One state_id was wrong.
   - `sensor.metering_total_absorbed` has correct inserts for the timespan.
   - `sensor.metering_total_yield_compensation` - One state_id was wrong.
 
To update both `state` and `sum` column, I copied the data into a excel-matrix and made sql-commands based on the data, such as `update homeassistant.statistics_short_term set sum = 604.883 where id = 33890;` and `update homeassistant.states set state = 2756.716 where state_id = 1289802;`.
I did not utilize the python program suggested in above links.

#### For the InfluxDB history database, the following was performed:

Since the data in the `states` table is wrong, we can assume that the data is also wrong in the InfluxDB database as states are pushed here.

We cannot correct the data in InfluxDB directly through commands due to the design of the time-series database, instead we import corrected data so the data is overwritten.

Be extra cautious with the delete command, preferably take a backup before updating.

1. Logon to InfluxDB databas and run the following in `Data Explorer` (Table view) for the above identified sensors and the valid time-ranges, according to:
```
from(bucket: "ha")
  |> range(start: 2022-01-24T00:00:00Z, stop: 2022-01-26T12:00:00Z)
  |> filter(fn: (r) => r["entity_id"] == "total_yield")
```
2. Export each sensor to csv.
3. In excel, isolate if there are wrong data during the timespan.
   - In my case, `_value` was 0 (zero) for `_time` 2022-01-25T00:42:43.152029Z for `total_yield`.
   - And similar for the following entity_ids:
     - `pv_gen_meter`.
     - `metering_total_yield`.
     - `metering_total_yield_compensation`.
5. Go to the InfluxDB container on server1:
   - `sudo docker-compose exec ha-history-db bash`.
     - With shell in the container, delete the spefic _time (I did not manage to overwrite data with export/import-csv):
       - Note that you need to be precise with the timestamp, as we do not use '--predicate', and would otherwise delete extensive amount of data.
         ```
         influx delete -b ha --start '2022-01-25T00:42:43.152029Z' --stop '2022-01-25T00:42:43.152029Z'
         ```
         - No error/output should occur.
    - Iterate through all above sensors and correct where necessary.

### Incorrect Balboa Spa data

Sometime after 2022-12-08, after reboot of HA, due to upgrade, the sensor 'balboa_spa_heater_running_time_hour' went havoc.
It appears that it started to register time the heater was on, even though the underlying sensor was not 'on'.

After analysis it appears that the data was wrong between 2022-12-08 01:20 and 2022-12-15 08:05.

Since the sensor `balboa_spa_heater_running_time_hour` is also used by `balboa_spa_heater_consumption_hour` and `balboa_spa_circulation_pump_heater_consumption_hour` these are also wrong (note that `balboa_spa_circulationpump_consumption_hour` was not wrong).

#### For Recorder database (MariaDB) i am using MySQL Workbench and excel to correct:

Since the database structure has been changed a few version back (at writing, the version is 2022.11.2), we cannot reuse all from the error with SMA-data.

We first need to isolate the right `metadata_id` for the sensors from table `statistics_short_term`:
- Meta data id 173 = `balboa_spa_heater_running_time_hour`
- Meta data id 151 = `balboa_spa_heater_consumption_hour`
- Meta data id 163 = `balboa_spa_circulation_pump_heater_consumption_hour`

After that we can delete data from the statistics tables `statistics_short_term` and `statistics` with the `metadata_id` isolated above.

#### For the InfluxDB history database, the following was performed:

Since the data in the `states` table is wrong, we can assume that the data is also wrong in the InfluxDB database as states are pushed here.

We cannot correct the data in InfluxDB directly through commands due to the design of the time-series database, instead we import corrected data so the data is overwritten.

Be extra cautious with the delete command, preferably take a backup before updating.
(I tried `--predicate 'entity_id="balboa_spa_circulation_pump_heater_consumption_hour"'` but it did not delete data. Bug?)

1. Logon to InfluxDB databas and run the following in `Data Explorer` (Table view) for the above identified sensors and the valid time-ranges, according to:
```
from(bucket: "ha")
  |> range(start: 2022-12-08T01:20:00Z, stop: 2022-12-15T08:05:00Z)
  |> filter(fn: (r) => r["entity_id"] == "balboa_spa_heater_consumption_hour" and r["_field"] == "value")
```
2. Export each sensor to csv.
3. In excel, isolate the timespan that is wrong.
   - And similar for the following entity_ids:
     - `balboa_spa_heater_running_time_hour`.
     - `balboa_spa_circulation_pump_heater_consumption_hour`.
5. Go to the InfluxDB container on server1:
   - `sudo docker-compose exec ha-history-db bash`.
     - With shell in the container, delete the spefic _time (I did not manage to overwrite data with export/import-csv):
       - Note that you need to be precise with the timestamp, as we do not use '--predicate', and would otherwise delete extensive amount of data.
         ```
         influx delete -b ha --start '2022-01-25T00:42:43.152029Z' --stop '2022-01-25T00:42:43.152029Z'
         ```
         - No error/output should occur.
    - Iterate through all above sensors and correct where necessary.
    - Note that this will take time as each commit will take 3-5 seconds.
