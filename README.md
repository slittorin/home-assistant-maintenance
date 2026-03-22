# Home Assistant - Maintenance

## Table of content

- Regular maintenance:
  - [Check updates for Home Assistant](https://github.com/slittorin/home-assistant-maintenance#check-updates-for-home-assistant)
  - [Check updates for server1](https://github.com/slittorin/home-assistant-maintenance#check-updates-for-server1)
  - [Check updates for InfluxDB](https://github.com/slittorin/home-assistant-maintenance#check-updates-for-influxdb)
  - [Check updates for Grafana](https://github.com/slittorin/home-assistant-maintenance#check-updates-for-grafana)
  - [Remove obsolete entities](https://github.com/slittorin/home-assistant-maintenance#remove-obsolete-or-faulty-entities)
  - [Remove data from Home Assistant database](https://github.com/slittorin/home-assistant-maintenance?tab=readme-ov-file#remove-data-from-home-assistant-database)
  - [Add domain sensors](https://github.com/slittorin/home-assistant-maintenance#add-domain-sensors)
  - [Exclude sensors for InfluxDB integration](https://github.com/slittorin/home-assistant-maintenance#exclude-sensors-for-influxdb-integration)
  - [Exclude sensors for InfluxDB integration - reduce InfluxDB size](https://github.com/slittorin/home-assistant-maintenance#exclude-sensors-for-influxdb-integration---reduce-influxdb-size)
- Errors, problems and challenges:
  - [Incorrect SMA Energy data](https://github.com/slittorin/home-assistant-maintenance#incorrect-sma-energy-data)
  - [Incorrect Balboa Spa data](https://github.com/slittorin/home-assistant-maintenance#incorrect-balboa-spa-data)
  - [Failed SSD drive](https://github.com/slittorin/home-assistant-maintenance/blob/main/README.md#failed-ssd-drive)
  - [After SSD disk crash, restore of data from MySQL to InfluxDB](https://github.com/slittorin/home-assistant-maintenance/blob/main/README.md#after-ssd-disk-crash-restore-of-data-from-mysql-to-influxdb)
  - [After SSD disk crash, restore of data from MySQL to InfluxDB - Restore failure](https://github.com/slittorin/home-assistant-maintenance/blob/main/README.md#after-ssd-disk-crash-restore-of-data-from-mysql-to-influxdb---restore-failure)
  - [After update and reboot of server1 caused the ha-database to be lost](https://github.com/slittorin/home-assistant-maintenance/blob/main/README.md#after-update-and-reboot-of-server1-caused-the-ha-database-to-be-lost)

## Regular maintenance

### Check updates for Home Assistant

Check regularly if there is any update for Home Assistant: Core, Operation System or Supervisor.\
Install through the UI or through manual update [link](https://www.home-assistant.io/common-tasks/os/#run-a-specific-version).\
`ha core update --backup --version VERSION`\
Note that you are encoruaged to take backup prior to any upgrade.

### Check updates for server1

Check regularly if there is any update for server1 (Raspberry PI).\
Install through normal `apt` commands.

### Check updates for InfluxDB

Check regularly if there is any important update that is required for InfluxDB docker-container.\
Note that you are encouraged to take backup prior to any upgrade.

1. Isolate current image-versions by login to server1 and running the command `sudo docker container ls`.
   - The result at 9/1-2024 is `influxdb:2.2.0-alpine`. 
2. See release information [here](https://docs.influxdata.com/influxdb/v2/reference/release-notes/influxdb/) and [here](https://github.com/influxdata/influxdb/releases).
3. Isolate if there is a need to upgrade the image, such as security, performance upgrades or bugs.
     - Bugs can be found [here](https://github.com/influxdata/influxdata-docker/issues?q=).
     - See also Reddit [threat](https://www.reddit.com/r/influxdb/comments/15l0cto/support_timeline_for_influxdb_2/) about life-span for InfluxDB2.
     - Look specifically for breaking changes.
     - If there is a need to upgrade, do this preferably in steps, and with full backup and verification of functionality in between.
3. Isolate the image to upgrade to.
4. Log on to server1:
   - Go to directory `/srv`.
   - Run a backup with `sudo ./influxdb-backup.sh`.
     - Verify that backup has been created in directory `/srv/ha-history-db/backup`, with a correct size (check past backups).
     - Check the last part of the logfile `/srv/log/influxdb-backup.log` that the backup was done correctly.
     - If there are errors, do not upgrade.
   - Check the image running for InfluxDB with `docker ps`.
   - Change and save the file `docker-compose.yml`.
     - For `ha-history-db`, change the line with `influxdb:2.2.0-alpine` (or similar) to the image chosen in step 3.
   - Allow docker-compose to pull the image with `sudo docker-compose pull`.
     - Ensure that the image is pulled, and that there are no warnings or errors.
   - Allow docker-compose to update update the docker with `sudo docker-compose up -d --remove-orphans`.
     - Ensure that there are no warnings or errors.
   - Check the image running for grafana with `docker ps` and that it is the right image.
5. Go to the InfluxDB web-interface and verify functionality.

### Check updates for Grafana

Check regularly if there is any important update that is required for Grafana docker-container.\
Note that you are encouraged to take backup prior to any upgrade.

1. Isolate current image-versions by login to server1 and running the command `sudo docker container ls`.
   - See release information [here](https://grafana.com/docs/grafana/latest/whatsnew/) and [here](https://grafana.com/docs/grafana/latest/release-notes/).
2. Isolate if there is a need to upgrade the image, such as security, performance upgrades or bugs.
   - Bugs can be found [here]((https://github.com/grafana/grafana/issues).
   - Breaking changes can be found [here](https://grafana.com/docs/grafana/latest/breaking-changes/).
   - Further information can be found [here](https://grafana.com/docs/grafana/latest/whatsnew/whats-new-in-v9-0/) (go to the correct version).
   - If there is a need to upgrade, do this preferably in steps, and with full backup and verification of functionality in between.
3. Isolate the image to upgrade to.
4. Log on to server1:
   - Go to directory `/srv`.
   - Run a backup with `sudo ./grafana-backup.sh`.
     - Verify that backup has been created in directory `/srv/ha-grafana/backup`.
     - Check the last part of the logfile `/srv/log/grafana-backup.log` that the backup was done correctly, with a correct size (check past backups).
     - If there are errors, do not upgrade.
   - Check the image running for Grafana with `docker ps`.
   - Change and save the file `docker-compose.yml`.
     - For `ha-grafana`, change the line with `image: grafana/grafana:8.3.3` (or similar) to the image chosen in step 3.
   - Allow docker-compose to pull the image with `sudo docker-compose pull`.
     - Ensure that the image is pulled, and that there are no warnings or errors.
   - Allow docker-compose to update update the docker with `sudo docker-compose up -d --remove-orphans`.
     - Ensure that there are no warnings or errors.
   - Check the image running for grafana with `docker ps` and that it is the right image.
5. Go to the Grafana web-interface and verify functionality.

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

### Remove data from Home Assistant database

Install Add-On for phpMyAdmin, or connect to the database from externally.

Utilize with caution, always take backup before deleting.

We utilize the examples below for the following sensor: `sensor.balboa_spa_circulation_pump_heater_consumption_hour`

#### Remove data from Home Assistant state table

1. Isolate the right metadata_id
Utilize the followin SQL to isolate the right metadata_id:
```sql
SELECT *
FROM homeassistant.states_meta
WHERE entity_id = 'sensor.balboa_spa_circulation_pump_heater_consumption_hour';
```
Take note of the value for `metadata_id` `(A)`, in this case it is `270`.

2. Isolate the right data for the specific time period:
Set the value for `metadata_id` to the `(A)` value.
Update the limit to appropriate value if data is not found.
```sql
SELECT *
FROM homeassistant.states
WHERE metadata_id = 270
  AND last_updated_ts BETWEEN UNIX_TIMESTAMP('2026-03-02 00:00:00') AND UNIX_TIMESTAMP('2026-03-03 23:59:59')
LIMIT 1000;
```
Isolate the value for `state_id` `(B)`, in this case it is `607603044`. This is the id to be deleted.\
Isolate the value for `old_state_id` `(C)`, in this case it is `607541794`. This is the id we need to not break the index.

3. Verify the data before deletion:
Set the value for `state_id` to the `(B)` value.
```sql
SELECT *
FROM homeassistant.states
WHERE state_id = 607603044;
```
If the row presented is the correct one to delete, proceed with the below.

4. We need to get the row that references the id to be deleted:
Set the value for `old_state_id` to the `(B)` value.
```sql
SELECT *
FROM homeassistant.states
WHERE old_state_id = 607603044;
```
Take note of the value for `state_id` `(D)`, in this case it is `607632453`. This is the id we need to update for `old_state_id`.

5. We need to update so that the index do not break:
Set the value for `old_state_id` to the `(C)` value.\
Set the value for `state_id` to the `(D)` value.
```sql
UPDATE homeassistant.states
SET old_state_id = 607541794
WHERE state_id = 607632453;
```

6. We thereafter can delete the state:
Set the value for `state_id` to the `(B)` value.
```sql
DELETE FROM homeassistant.states
WHERE state_id = 607603044;
```

#### Remove data from Home Assistant statistics table

1. Isolate the right metadata_id
Utilize the followin SQL to isolate the right id:
```sql
SELECT *
FROM homeassistant.statistics_meta
WHERE statistic_id = 'sensor.balboa_spa_circulation_pump_heater_consumption_hour';
```
Take note of the value for `id` `(E)`, in this case it is `163`.

2. Isolate the right data for the specific time period:
Set the value for `metadata_id` to the `(E)` value.
Update the limit to appropriate value if data is not found.
```sql
SELECT *
FROM homeassistant.statistics
WHERE metadata_id = 163
  AND created_ts BETWEEN UNIX_TIMESTAMP('2026-03-02 00:00:00') AND UNIX_TIMESTAMP('2026-03-03 23:59:59')
LIMIT 1000;
```
Isolate the value for `id` `(F)`, in this case it is `8073434`. This is the id to be deleted.

3. Verify the data before deletion:
Set the value for `id` to the `(F)` value.
```sql
SELECT *
FROM homeassistant.statistics
WHERE id = 8073434;
```
If the row presented is the correct one to delete, proceed with the below.

4. We thereafter can delete the statistics-id:
Set the value for `id` to the `(F)` value.
```sql
DELETE FROM homeassistant.statistics
WHERE id = 8073434;
```

#### Remove data from Home Assistant statistics_short_term table

Note that sum-values in the table is not updated with below method.

1. Isolate the right data for the specific time period:
Set the value for `metadata_id` to the `(E)` value.
Update the limit to appropriate value if data is not found.
```sql
SELECT *
FROM homeassistant.statistics_short_term
WHERE metadata_id = 163
  AND created_ts BETWEEN UNIX_TIMESTAMP('2026-03-02 00:00:00') AND UNIX_TIMESTAMP('2026-03-03 23:59:59')
LIMIT 1000;
```
Isolate the value(s) for `id` `(G)`, in this case it is `96114730`. This is the id to be deleted.

3. Verify the data before deletion:
Set the value for `id` to the `(G)` value.
```sql
SELECT *
FROM homeassistant.statistics_short_term
WHERE id = 96114730;
```
If the row presented is the correct one to delete, proceed with the below.

4. We thereafter can delete the statistics_short_term-id:
Set the value for `id` to the `(G)` value.
```sql
DELETE FROM homeassistant.statistics_short_term
WHERE id = 96114730;
```
#### Update sum-column in Home Assistant statistics table

When removing data in the statistics table, the summarized data in column `sum` does not update.\
And this value is used for Home Assistant and by graphs, energy dashboards, utility meters, etc.

Therefore we need to iterate through the table and update the value in column `sum`.

To do this, you need to:
- Retrieve the value `(E)` from above.
- Identify the first row with the wrong data and retrieve the value for column `created_ts`.
- Identify the last row with the wrong data and retrieve the value for column `created_ts`.
- Identify the value you want to correct the values in column `sum` with.
  - If you want to add 501.2, you use the value: 501.2
  - If you want to remove 103.33, you use the value: -103.33

Always ensure that you have backed up the MariaDB-database before running the stored procedure.\
Always ensure that you have verified the values to add into the stored procedure.\
Always ensure that you run the script when the sensor is not updated (in this case it runs once each hour).

First run the stored procedure with (example):
```sql
CALL homeassistant.adjust_statistics_sum(163, 1772463611.7957425, 1774087211.5659456, -706.50162, 'info');
```
Look at the output and enumerate so that the correct data are shown for: `id, created_ts, old_sum, new_sum`.

If the values are correct, you can run the stored procedure with (example):
```sql
CALL homeassistant.adjust_statistics_sum(163, 1772463611.7957425, 1774087211.5659456, -706.50162, 'update');
```
Look at the output and enumerate so that the correct data are shown for: `id, created_ts, old_sum, new_sum`.

Create the stored procedure with the below:
```sql
-- =============================================================================
-- Stored Procedure: adjust_statistics_sum
-- Database:         homeassistant
-- Table:            statistics
--
-- Purpose:          Adjusts the 'sum' column for rows matching a given
--                   metadata_id within a created_ts range (inclusive).
--                   Supports dry-run (info) and live update modes.
--
-- Usage examples:
--   Dry-run (preview only):
--     CALL homeassistant.adjust_statistics_sum(6, 1774058000.0, 1774059000.0, -3.5, 'info');
--
--   Live update:
--     CALL homeassistant.adjust_statistics_sum(6, 1774058000.0, 1774059000.0, -3.5, 'update');
-- =============================================================================

DELIMITER $$

DROP PROCEDURE IF EXISTS homeassistant.adjust_statistics_sum$$

CREATE PROCEDURE homeassistant.adjust_statistics_sum(
    IN p_metadata_id   INT,
    IN p_from_ts       DOUBLE,
    IN p_to_ts         DOUBLE,
    IN p_adjustment    DOUBLE,
    IN p_mode          VARCHAR(10)   -- 'update' or 'info'
)
BEGIN
    -- -----------------------------------------------------------------
    -- Variable declarations (must come before any executable statements)
    -- -----------------------------------------------------------------
    DECLARE v_id           INT;
    DECLARE v_created_ts   DOUBLE;
    DECLARE v_old_sum      DOUBLE;
    DECLARE v_new_sum      DOUBLE;
    DECLARE v_row_count    INT DEFAULT 0;
    DECLARE v_done         INT DEFAULT 0;

    -- Cursor: selects the rows that match the criteria
    DECLARE cur CURSOR FOR
        SELECT id, created_ts, `sum`
        FROM   homeassistant.statistics
        WHERE  metadata_id = p_metadata_id
          AND  created_ts >= p_from_ts
          AND  created_ts <= p_to_ts
        ORDER  BY created_ts;

    DECLARE CONTINUE HANDLER FOR NOT FOUND SET v_done = 1;

    -- -----------------------------------------------------------------
    -- Input validation
    -- -----------------------------------------------------------------
    IF LOWER(p_mode) NOT IN ('update', 'info') THEN
        SELECT '*** ERROR: p_mode must be ''update'' or ''info''.' AS message;
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'Invalid mode. Use ''update'' or ''info''.';
    END IF;

    IF p_from_ts > p_to_ts THEN
        SELECT '*** ERROR: p_from_ts is greater than p_to_ts.' AS message;
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'from-timestamp must be <= to-timestamp.';
    END IF;

    -- -----------------------------------------------------------------
    -- Header / summary info
    -- -----------------------------------------------------------------
    SELECT '============================================================' AS info
    UNION ALL
    SELECT CONCAT('  Procedure     : adjust_statistics_sum')
    UNION ALL
    SELECT CONCAT('  Mode          : ', UPPER(p_mode),
                  CASE WHEN LOWER(p_mode) = 'info'
                       THEN '  (dry-run, no changes will be made)'
                       ELSE '  (LIVE — rows will be updated)'
                  END)
    UNION ALL
    SELECT CONCAT('  metadata_id   : ', p_metadata_id)
    UNION ALL
    SELECT CONCAT('  from_ts       : ', p_from_ts)
    UNION ALL
    SELECT CONCAT('  to_ts         : ', p_to_ts)
    UNION ALL
    SELECT CONCAT('  adjustment    : ', p_adjustment)
    UNION ALL
    SELECT '============================================================';

    -- -----------------------------------------------------------------
    -- Create a temporary table for collecting per-row results
    -- -----------------------------------------------------------------
    DROP TEMPORARY TABLE IF EXISTS _tmp_adjust_results;
    CREATE TEMPORARY TABLE _tmp_adjust_results (
        row_num    INT AUTO_INCREMENT PRIMARY KEY,
        id         INT,
        created_ts DOUBLE,
        old_sum    DOUBLE,
        new_sum    DOUBLE,
        status     VARCHAR(20)
    );

    -- -----------------------------------------------------------------
    -- Iterate over matching rows
    -- -----------------------------------------------------------------
    OPEN cur;

    read_loop: LOOP
        FETCH cur INTO v_id, v_created_ts, v_old_sum;
        IF v_done THEN
            LEAVE read_loop;
        END IF;

        SET v_new_sum  = v_old_sum + p_adjustment;
        SET v_row_count = v_row_count + 1;

        -- Perform the actual UPDATE only in 'update' mode
        IF LOWER(p_mode) = 'update' THEN
            UPDATE homeassistant.statistics
            SET    `sum` = v_new_sum
            WHERE  id = v_id;

            INSERT INTO _tmp_adjust_results (id, created_ts, old_sum, new_sum, status)
            VALUES (v_id, v_created_ts, v_old_sum, v_new_sum, 'UPDATED');
        ELSE
            INSERT INTO _tmp_adjust_results (id, created_ts, old_sum, new_sum, status)
            VALUES (v_id, v_created_ts, v_old_sum, v_new_sum, 'DRY-RUN');
        END IF;

    END LOOP;

    CLOSE cur;

    -- -----------------------------------------------------------------
    -- Output the per-row results
    -- -----------------------------------------------------------------
    SELECT id,
           created_ts,
           old_sum,
           new_sum,
           (new_sum - old_sum) AS applied_adjustment,
           status
    FROM   _tmp_adjust_results
    ORDER  BY row_num;

    -- -----------------------------------------------------------------
    -- Footer / totals
    -- -----------------------------------------------------------------
    SELECT CONCAT('Total rows matched: ', v_row_count) AS summary
    UNION ALL
    SELECT CONCAT('Adjustment value : ', p_adjustment)
    UNION ALL
    SELECT CONCAT('Action taken     : ',
                  CASE WHEN LOWER(p_mode) = 'update'
                       THEN CONCAT(v_row_count, ' row(s) UPDATED')
                       ELSE 'None (dry-run)'
                  END);

    DROP TEMPORARY TABLE IF EXISTS _tmp_adjust_results;

END$$

DELIMITER ;
```

### Add domain sensors

With they way we are tracking data, we need add sensors when we add integrations/add-ons/devices to our HA system.

1. Keep track if there are new domains that are not tracked.
   - Go to Dashboard Home Assistant.
   - Check the listed domains towards those documented in [Domains and Entities configuration](https://github.com/slittorin/home-assistant-configuration#package---home-assistant-system---domains-and-entities).
2. Where required, add domains and sensors.
   - Update also [Visualization for Number of domains and entities](https://github.com/slittorin/home-assistant-visualization#number-of-domains-and-entities).

### Exclude sensors for InfluxDB integration

See also 'Exclude sensors for InfluxDB integration - reduce InfluxDB size' below.

#### Isolate possible sensors by looking into Home Assistant database

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

#### Isolate possible sensors by looking into InfluxDB database

I created the below Flux-script in Grafana, and added it to a separate view in Home Assistant.

This allows me to get statistics of most sensors in InfluxDB for the last 10 years.\
And allows me to identify sensors that fill upp space, and are not valid from historical or statistical purpose.\
That can thereafter be removed from the InfluxDB-database (see further below).

I also created a 30 day view, to allow me to see the latest data as well.

The script shows me the following data for top 100 entities:
- first_time seen in the InfluxDB database for the stated timeframe.
- last_time seen in the InfluxDB database for the stated timeframe.
- domain in Home Assistant.
- entity_id in Home Assistant.
- percentage in the InfluxDB database for the stated timeframe.
- row_count in the InfluxDB database for the stated timeframe.

```
counts =
  from(bucket: "ha")
    |> range(start: -10y)
    |> filter(fn: (r) => exists r["entity_id"])
    |> filter(fn: (r) => r["_field"] == "state")
    |> group(columns: ["entity_id", "domain"])    // ← group on both
    |> count()
    |> keep(columns: ["entity_id", "domain", "_value"])
    |> group(columns: [])

total =
  counts
    |> sum(column: "_value")
    |> findRecord(fn: (key) => true, idx: 0)

first_times =
  from(bucket: "ha")
    |> range(start: -10y)
    |> filter(fn: (r) => exists r["entity_id"])
    |> filter(fn: (r) => r["_field"] == "state")
    |> group(columns: ["entity_id"])
    |> first()
    |> keep(columns: ["entity_id", "_time"])
    |> rename(columns: {_time: "first_time"})
    |> group(columns: [])

last_times =
  from(bucket: "ha")
    |> range(start: -10y)
    |> filter(fn: (r) => exists r["entity_id"])
    |> filter(fn: (r) => r["_field"] == "state")
    |> group(columns: ["entity_id"])
    |> last()
    |> keep(columns: ["entity_id", "_time"])
    |> rename(columns: {_time: "last_time"})
    |> group(columns: [])

with_percent =
  counts
    |> map(fn: (r) => ({
        r with
        row_count: r._value,
        percent:   100.0 * float(v: r._value) / float(v: total._value)
      }))
    |> keep(columns: ["entity_id", "domain", "row_count", "percent"])  // ← keep domain

join(
  tables: {
    a: join(
         tables: {base: with_percent, ft: first_times},
         on: ["entity_id"]
       ),
    lt: last_times
  },
  on: ["entity_id"]
)
  |> keep(columns: ["entity_id", "domain", "row_count", "percent", "first_time", "last_time"])
  |> sort(columns: ["row_count"], desc: true)
  |> limit(n: 100)
```

### Exclude sensors for InfluxDB integration - reduce InfluxDB size

#### Downsample sensor-data

Always ensure that you have backed up the InfluxDB before running scripts.\
Always ensure that you have verified the values to utilize for the scripts.

My InfluxDB quickly increased in size, up until approx 1030 MB in early 2023.

After analysis of the data received into InfluxDB over a month, I isolated a number of further entities that filled up the database.
Here i logged into Influx and did an export to CSV, and utilized a pivot-table to isolate the sensors that gave alot of data.

I here isolated that:
- I had at most 120K state-events per day was entered into the database(2022-11-17).
- Even after reduction made in December 2022, I still had 40K state-events per day entered into the database.
- I had over a year accumulated 38M state-events.
- Each state-event, also adds 3-4 other measures (name, class and so forth), so the volume above is actually times 3 to 4.

I isolated a bunch of further sensors (see [configuration.yaml](https://github.com/slittorin/home-assistant-config/blob/master/configuration.yaml)), that we downsampled further (see [Packages](https://github.com/slittorin/home-assistant-config/tree/master/packages)).

Once these where running correctly with down-sampling, we are down to 4K-7K state-events per day.

Now we needed to figure out how to remove data in Influx, and at the same time have hourly data.

I came up with the below approach, including scripts that would ease the effort (yes, I know, if I had written in Python (or other) they would have been quicker to run).
The convert-script can take max, min, average, last, first and sum for each hour for a sensor/entity.

1. Isolate when the original-sensor started recording, and when it ended.
   - See [Flux language](https://github.com/slittorin/home-assistant-visualization#generic-information).
2. Isolate when down-sampled started, utilize same Flux language as above.
3. Run script [influxdb-convert-to-hourly.sh](https://github.com/slittorin/server1-config/blob/master/influxdb-convert-to-hourly.sh) to get the hourly data exported as csv-files.
   - Example: `sudo /srv/influxdb-convert-to-hourly.sh -f 20220118 -t 20220216 -e metering_current_l1 -n sma_metering_current_l1_min_hour -c min -d 4`.
4. Verify the log and output. Including to remove any hourly-data in last csv-file, so it does not overlap with existing sensor/entity-data.
5. Import the data into Flux with script [influxdb-import.sh](https://github.com/slittorin/server1-config/blob/master/influxdb-import.sh).
   - Example: `sudo /srv/influxdb-import.sh -f 20220118 -t 20221110 -e sma_metering_power_supplied_min_hour`.
6. Verify that the data is correctly written in to Influx. Use Grafana or Influx to verify this.
7. Once we have all necessary hourly-data in InfluxDB, we can remove the data.
   - Here we log into the Influx-container with `sudo docker-compose exec ha-history-db bash`.
   - And then run delete commands, such as: `influx delete --bucket ha --start '2022-01-18T00:00:00.000Z' --stop '2022-02-17T00:01:00.000Z' --predicate 'entity_id="metering_current_l1"'`.
   - Note that it is important to use the right entity_id.
8. Verify that the data is correctly deleted from Influx. Use Grafana or Influx to verify this.

I managed to the InfluxDB down from 1030MB to 130MB by doing this, and also reduced the load/daily-increase in size.
A benefit is also that the load-time for some of the graphs has reduced 5-10 folds, as there is only hourly-data to collect.

#### Remove specific sensor-data entries/rows

Always ensure that you have backed up the InfluxDB before running scripts.\
Always ensure that you have verified the values to utilize for the scripts.

This can be used to remove specific sensor data in InfluxDB.

First, export the data from InfluxDB by running the following script on server1 in directory `/srv`:
```
sudo ./influxdb-entity-export.sh -e sensor.balboa_spa_circulation_pump_heater_consumption_hour -f 20260321 -t 20260322
```
Change the name of the sensor (-e), and the from (-f) and to (-t) dates.

The output looks like the following:
```
Finished InfluxDB entity export. No error.
CSV-file output: /srv/ha-history-db/export/influx-entity-export-sensor_balboa_spa_circulation_pump_heater_consumption_hour-20260321-20260322.csv
Update file and keep only rows that are to be deleted.
Run thereafter the following script: influxdb-entity-delete.sh -i /srv/ha-history-db/export/influx-entity-export-sensor_balboa_spa_circulation_pump_heater_consumption_hour-20260321-20260322.csv
You can thereafter run this script again to export the data. Note however that file can be overwritten!
```
Log-data for the script exists in directory `/srv/log`.

Second, update the exported file to ONLY include rows that are to be deleted from InfluxDB.

Thirdly, run the following script in directory `/srv` (note that it can take long time to delete):
```
sudo influxdb-entity-delete.sh -i /srv/ha-history-db/export/influx-entity-export-sensor_balboa_spa_circulation_pump_heater_consumption_hour-20260321-20260322.csv
```
Log-data for the script exists in directory `/srv/log`.

Forthly, rename the file csv-file to a new name, so we know what was deleted.

Fiftly, export the data from InfluxDB again by running the following script on server1 in directory `/srv`:
```
sudo ./influxdb-entity-export.sh -e sensor.balboa_spa_circulation_pump_heater_consumption_hour -f 20260321 -t 20260322
```
Look thereafter into the csv-file and isolate if the rows/data where deleted.

#### Remove sensor-data for a date-range.

Always ensure that you have backed up the InfluxDB before running scripts.\
Always ensure that you have verified the values to utilize for the scripts.

This can be used to remove sensordata in InfluxDB for a specific date-range.

First, export the data from InfluxDB by running the following script on server1 in directory `/srv`:
```
sudo ./influxdb-entity-export.sh -e binary_sensor.ehs4plem_online -f 20220601 -t 20220701
```
Change the name of the sensor (-e), and the from (-f) and to (-t) dates.
Log-data for the script exists in directory `/srv/log`.

Secondly, run the following script in directory `/srv` (note that it can take long time to delete):
```
sudo ./influxdb-entity-delete-daterange.sh -e binary_sensor.ehs4plem_online -f 20220601 -t 20220701
```
Log-data for the script exists in directory `/srv/log`.

The output looks like the following:
```
=====================================================================
 DATA TO BE DELETED FROM INFLUXDB
=====================================================================
 Entity ID  : binary_sensor.ehs4plem_online
 Predicate  : entity_id="ehs4plem_online"
 Start      : 2022-06-01T00:00:00.000000Z
 Stop       : 2022-07-01T23:59:59.999999Z
 Bucket     : ha
 Org        : lite
=====================================================================

CAUTION: ALL data points for this entity in the given date range will
         be permanently deleted from InfluxDB.
         This operation CANNOT be undone.

Tip: Run influxd-entity-export.sh first to preview the data if you
     have not already done so.

Type YES (uppercase) to confirm deletion, or anything else to abort: YES

Confirmed. Starting deletion...

Finished InfluxDB entity range-delete. No error.

IMPORTANT: influx delete does not confirm how many data points were actually
removed. Always verify the result by re-running influxd-entity-export.sh
for the same entity and date range.
```

Thirdly, check that the data is deleted, either through the Grafana-view listed above, or through a new export.

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
     - With shell in the container, delete the specific _time (I did not manage to overwrite data with export/import-csv):
       - Note that you need to be precise with the timestamp, as we do not use '--predicate', and would otherwise (potentially) delete extensive amount of data.
         ```
         influx delete -b ha --start '2022-01-25T00:42:43.152029Z' --stop '2022-01-25T00:42:43.152029Z'
         ```
         - No error/output should occur.
    - Iterate through all above sensors and correct where necessary.

### Incorrect Balboa Spa data

Sometime after 2022-12-08, after reboot of HA, due to upgrade, the sensor 'balboa_spa_heater_running_time_hour' went havoc.
It appears that it started to register time the heater was on, even though the underlying sensor was not 'on'.

After analysis it appears that the data was wrong between 2022-12-08 01:20 and 2022-12-15 08:05.

Since the sensor `balboa_spa_heater_running_time_hour` is also used by `balboa_spa_heater_consumption_hour`, `balboa_spa_circulation_pump_heater_consumption_hour`, `balboa_spa_heater_cost_hour` and `balboa_spa_circulationpump_heater_cost_hour` these are also wrong (note that `balboa_spa_circulationpump_consumption_hour` was not wrong).

We therefore need to delete data (we cannot restore/correct any data).

#### For Recorder database (MariaDB) i am using MySQL Workbench and excel to correct:

Since the database structure has been changed a few version back (at writing, the version is 2022.11.2), we cannot reuse all from the error with SMA-data.

We first need to isolate the right `metadata_id` for the sensors from table `statistics_short_term`:
- `metadata_id` 173 = `balboa_spa_heater_running_time_hour`
- `metadata_id` 151 = `balboa_spa_heater_consumption_hour`
- `metadata_id` 163 = `balboa_spa_circulation_pump_heater_consumption_hour`
- `metadata_id` 152 = `balboa_spa_heater_cost_hour`
- `metadata_id` 161 = `balboa_spa_circulationpump_heater_cost_hour`

After that we can delete data from the statistics tables `statistics_short_term` and `statistics` with the `metadata_id` isolated above, either by `id` or datetime-range.

#### For the InfluxDB history database, the following was performed:

Since the data in the `states` table is wrong, we can assume that the data is also wrong in the InfluxDB database as states are pushed here.

We cannot correct the data in InfluxDB directly through commands due to the design of the time-series database, instead we import corrected data so the data is overwritten.

Be extra cautious with the delete command, preferably take a backup before updating.

1. Logon to InfluxDB databas and run the following in `Data Explorer` (Table view) for the above identified sensors and the valid time-ranges, according to:
```
from(bucket: "ha")
  |> range(start: 2022-12-08T01:20:00Z, stop: 2022-12-15T08:05:00Z)
  |> filter(fn: (r) => r["entity_id"] == "balboa_spa_heater_consumption_hour" and r["_field"] == "value")
```
   - And similar for the following entity_ids:
     - `balboa_spa_heater_running_time_hour`.
     - `balboa_spa_circulation_pump_heater_consumption_hour`.
     - `balboa_spa_heater_cost_hour`.
     - `balboa_spa_circulationpump_heater_cost_hour`.
2. Export each sensor to csv.
3. In excel, isolate the timespan that is wrong.
5. Go to the InfluxDB container on server1:
   - `sudo docker-compose exec ha-history-db bash`.
     - With shell in the container, delete the specific `entity_id` with start and stop according to first and latest according to export above.
       ```
       influx delete -b ha --start '2022-12-08T02:00:47.029684Z' --stop '2022-12-15T08:00:02.017015Z' --predicate 'entity_id="balboa_spa_heater_consumption_hour"'
       ```
       - No error/output should occur.
    - Iterate through all above sensors and correct where necessary.

### Failed SSD drive

#### Error

Between 4 and 5/1-2023 InfluxDB stopped and did not store any data.\
I noticed this 8/1 and tried to logon to RPI with SSH without luck.\
Interestingly, grafana-web was still working, and I could login to Influx also.

For login to RPI, I could enter username and password, but the login stopped right after pressing Enter.\
This occured both through SSH and in GUI.\
After a while, errors occured on GUI that was similar to [this](https://forums.raspberrypi.com/viewtopic.php?t=336457).

#### Analysis

After mounting the SSD drive on an Ubuntu-machine, and later on the RPI booted from the initial SD-card, I could determine that there was several errors for the SSD.

This was established through `fsck` for the device, where however it only found a few errors to correct.\
Noticable was also that the errors changed between running the command, and in many cases `fsck` did not state any errors.

It was only when I utilized `sudo smartctl -d sat -a /dev/sdb` (on my Ubuntu-machine, smartctl is not installed on RPI) I could see the extent of the problem:
```
=== START OF INFORMATION SECTION ===
Device Model:     Samsung SSD 870 EVO 500GB
Serial Number:    S6PYNF0RB03411L
LU WWN Device Id: 5 002538 f41b14378
Firmware Version: SVT01B6Q
User Capacity:    500 107 862 016 bytes [500 GB]
Sector Size:      512 bytes logical/physical
Rotation Rate:    Solid State Device
Form Factor:      2.5 inches
Device is:        Not in smartctl database [for details use: -P showall]
ATA Version is:   ACS-4 T13/BSR INCITS 529 revision 5
SATA Version is:  SATA 3.3, 6.0 Gb/s (current: 6.0 Gb/s)
Local Time is:    Mon Jan  9 22:29:54 2023 CET
SMART support is: Available - device has SMART capability.
SMART support is: Enabled

=== START OF READ SMART DATA SECTION ===
SMART overall-health self-assessment test result: PASSED

General SMART Values:
Offline data collection status:  (0x00)	Offline data collection activity
					was never started.
					Auto Offline Data Collection: Disabled.
Self-test execution status:      (   0)	The previous self-test routine completed
					without error or no self-test has ever 
					been run.
Total time to complete Offline 
data collection: 		(    0) seconds.
Offline data collection
capabilities: 			 (0x53) SMART execute Offline immediate.
					Auto Offline data collection on/off support.
					Suspend Offline collection upon new
					command.
					No Offline surface scan supported.
					Self-test supported.
					No Conveyance Self-test supported.
					Selective Self-test supported.
SMART capabilities:            (0x0003)	Saves SMART data before entering
					power-saving mode.
					Supports SMART auto save timer.
Error logging capability:        (0x01)	Error logging supported.
					General Purpose Logging supported.
Short self-test routine 
recommended polling time: 	 (   2) minutes.
Extended self-test routine
recommended polling time: 	 (  85) minutes.
SCT capabilities: 	       (0x003d)	SCT Status supported.
					SCT Error Recovery Control supported.
					SCT Feature Control supported.
					SCT Data Table supported.

SMART Attributes Data Structure revision number: 1
Vendor Specific SMART Attributes with Thresholds:
ID# ATTRIBUTE_NAME          FLAG     VALUE WORST THRESH TYPE      UPDATED  WHEN_FAILED RAW_VALUE
  5 Reallocated_Sector_Ct   0x0033   043   043   010    Pre-fail  Always       -       338
  9 Power_On_Hours          0x0032   098   098   000    Old_age   Always       -       8830
 12 Power_Cycle_Count       0x0032   099   099   000    Old_age   Always       -       46
177 Wear_Leveling_Count     0x0013   099   099   000    Pre-fail  Always       -       4
179 Used_Rsvd_Blk_Cnt_Tot   0x0013   043   043   010    Pre-fail  Always       -       338
181 Program_Fail_Cnt_Total  0x0032   100   100   010    Old_age   Always       -       0
182 Erase_Fail_Count_Total  0x0032   100   100   010    Old_age   Always       -       0
183 Runtime_Bad_Block       0x0013   043   043   010    Pre-fail  Always       -       338
187 Reported_Uncorrect      0x0032   099   099   000    Old_age   Always       -       2638
190 Airflow_Temperature_Cel 0x0032   068   053   000    Old_age   Always       -       32
195 Hardware_ECC_Recovered  0x001a   199   199   000    Old_age   Always       -       2638
199 UDMA_CRC_Error_Count    0x003e   100   100   000    Old_age   Always       -       0
235 Unknown_Attribute       0x0012   099   099   000    Old_age   Always       -       44
241 Total_LBAs_Written      0x0032   099   099   000    Old_age   Always       -       3168011128

SMART Error Log Version: 1
ATA Error Count: 2675 (device log contains only the most recent five errors)
	CR = Command Register [HEX]
	FR = Features Register [HEX]
	SC = Sector Count Register [HEX]
	SN = Sector Number Register [HEX]
	CL = Cylinder Low Register [HEX]
	CH = Cylinder High Register [HEX]
	DH = Device/Head Register [HEX]
	DC = Device Command Register [HEX]
	ER = Error register [HEX]
	ST = Status register [HEX]
Powered_Up_Time is measured from power on, and printed as
DDd+hh:mm:SS.sss where DD=days, hh=hours, mm=minutes,
SS=sec, and sss=millisec. It "wraps" after 49.710 days.

Error 2675 occurred at disk power-on lifetime: 8830 hours (367 days + 22 hours)
  When the command that caused the error occurred, the device was active or idle.

  After command completion occurred, registers were:
  ER ST SC SN CL CH DH
  -- -- -- -- -- -- --
  40 51 68 b8 7b 72 40  Error: UNC at LBA = 0x00727bb8 = 7502776

  Commands leading to the command that caused the error were:
  CR FR SC SN CL CH DH DC   Powered_Up_Time  Command/Feature_Name
  -- -- -- -- -- -- -- --  ----------------  --------------------
  60 00 68 b8 7b 72 40 0d      00:00:29.611  READ FPDMA QUEUED
  60 00 60 b8 77 72 40 0c      00:00:29.611  READ FPDMA QUEUED
  60 00 58 b8 73 72 40 0b      00:00:29.611  READ FPDMA QUEUED
  60 a8 50 10 72 72 40 0a      00:00:29.611  READ FPDMA QUEUED
  60 00 48 10 6e 72 40 09      00:00:29.611  READ FPDMA QUEUED

Error 2674 occurred at disk power-on lifetime: 8829 hours (367 days + 21 hours)
  When the command that caused the error occurred, the device was active or idle.

  After command completion occurred, registers were:
  ER ST SC SN CL CH DH
  -- -- -- -- -- -- --
  40 51 00 60 f8 8c 40  Error: UNC at LBA = 0x008cf860 = 9238624

  Commands leading to the command that caused the error were:
  CR FR SC SN CL CH DH DC   Powered_Up_Time  Command/Feature_Name
  -- -- -- -- -- -- -- --  ----------------  --------------------
  60 08 00 60 f8 8c 40 00      02:22:14.584  READ FPDMA QUEUED
  60 08 00 58 f8 8c 40 00      02:22:14.584  READ FPDMA QUEUED
  60 08 00 50 f8 8c 40 00      02:22:14.584  READ FPDMA QUEUED
  60 08 00 48 f8 8c 40 00      02:22:14.584  READ FPDMA QUEUED
  60 08 00 40 f8 8c 40 00      02:22:14.584  READ FPDMA QUEUED

Error 2673 occurred at disk power-on lifetime: 8829 hours (367 days + 21 hours)
  When the command that caused the error occurred, the device was active or idle.

  After command completion occurred, registers were:
  ER ST SC SN CL CH DH
  -- -- -- -- -- -- --
  40 51 08 58 aa 8c 40  Error: 

  Commands leading to the command that caused the error were:
  CR FR SC SN CL CH DH DC   Powered_Up_Time  Command/Feature_Name
  -- -- -- -- -- -- -- --  ----------------  --------------------
  ef 03 46 50 aa 8c 40 00      02:22:14.129  SET FEATURES [Set transfer mode]
  ef 03 0c 50 aa 8c 40 00      02:22:14.129  SET FEATURES [Set transfer mode]
  ec 08 00 50 aa 8c 40 00      02:22:14.129  IDENTIFY DEVICE
  60 08 00 50 aa 8c 40 00      02:22:14.129  READ FPDMA QUEUED
  60 08 00 48 aa 8c 40 00      02:22:14.129  READ FPDMA QUEUED

Error 2672 occurred at disk power-on lifetime: 8829 hours (367 days + 21 hours)
  When the command that caused the error occurred, the device was active or idle.

  After command completion occurred, registers were:
  ER ST SC SN CL CH DH
  -- -- -- -- -- -- --
  40 51 00 50 aa 8c 40  Error: UNC at LBA = 0x008caa50 = 9218640

  Commands leading to the command that caused the error were:
  CR FR SC SN CL CH DH DC   Powered_Up_Time  Command/Feature_Name
  -- -- -- -- -- -- -- --  ----------------  --------------------
  60 08 00 50 aa 8c 40 00      02:22:14.035  READ FPDMA QUEUED
  60 08 00 48 aa 8c 40 00      02:22:14.035  READ FPDMA QUEUED
  60 08 00 40 aa 8c 40 00      02:22:14.035  READ FPDMA QUEUED
  60 08 00 38 aa 8c 40 00      02:22:14.035  READ FPDMA QUEUED
  60 08 00 30 aa 8c 40 00      02:22:14.035  READ FPDMA QUEUED

Error 2671 occurred at disk power-on lifetime: 8829 hours (367 days + 21 hours)
  When the command that caused the error occurred, the device was active or idle.

  After command completion occurred, registers were:
  ER ST SC SN CL CH DH
  -- -- -- -- -- -- --
  40 51 00 b0 65 68 40  Error: UNC at LBA = 0x006865b0 = 6841776

  Commands leading to the command that caused the error were:
  CR FR SC SN CL CH DH DC   Powered_Up_Time  Command/Feature_Name
  -- -- -- -- -- -- -- --  ----------------  --------------------
  60 08 00 b0 65 68 40 00      02:21:35.854  READ FPDMA QUEUED
  60 08 00 a8 65 68 40 00      02:21:35.854  READ FPDMA QUEUED
  60 08 00 a0 65 68 40 00      02:21:35.854  READ FPDMA QUEUED
  60 08 00 98 65 68 40 00      02:21:35.854  READ FPDMA QUEUED
  60 08 00 90 65 68 40 00      02:21:35.854  READ FPDMA QUEUED

SMART Self-test log structure revision number 1
No self-tests have been logged.  [To run self-tests, use: smartctl -t]

SMART Selective self-test log data structure revision number 1
 SPAN  MIN_LBA  MAX_LBA  CURRENT_TEST_STATUS
    1        0        0  Not_testing
    2        0        0  Not_testing
    3        0        0  Not_testing
    4        0        0  Not_testing
    5        0        0  Not_testing
  256        0    65535  Read_scanning was never started
Selective self-test flags (0x0):
  After scanning selected spans, do NOT read-scan remainder of disk.
If Selective self-test is pending on power-up, resume after 0 minute delay.
```

I concluded that the SSD was so corrupted that it needed to be replaced.\

#### Recovery

Thereafter I proceeded to purchase the same drive (if the new one fails, then I must revisit my design and setup), including a new SATA to USB-cable, to ensure that there are no hardware errors that can affect the new setup.

At this stage I was also able to copy the `/srv` and `/var/lib/docker` libraries to the Ubuntu-machine.\
Note to self here to ensure that we have better copy of images and important configuration to another drive/share in the future, when drive fails (and yes, it will fail).

1. Since I had the old SD-card, I could easily setup the new SSD according to my [instructions](https://github.com/slittorin/rpi-boot-ssd).

2. Here I also attached the new SSD to my Ubuntu-machine and copied the saved `/srv` and `/var/lib/docker` to `/srv/restored`, so I could restore the files I needed.

3. After reboot of the RPI with the new SSD, I also ensured that the steps in [install](https://github.com/slittorin/raspberrypi-install) was performed.

4. Since the SD-card was before the actual configuration of Server1, I needed to perform the [installation](https://github.com/slittorin/home-assistant-setup#setup-for-server-1) steps again (with a fresh upgrade before).

5. Perform the setup [Preparation and step 1](https://github.com/slittorin/home-assistant-setup#preparation).

6. After that I copied the all the files and sub-dirs in directory `/srv/recovery/srv` to `/srv`, with (in dir /srv) `sudo cp -r restored/srv/* .`.\
   Here I verified that all the files had owner `xxx` and group `xxx`, and that shell-files could be run by user and group.

7. Setup [Preparation and step 2](https://github.com/slittorin/home-assistant-setup#preparation) was not needed to be performed since we already had the restored files.

8. Perform [Preparation and step 3](https://github.com/slittorin/home-assistant-setup#preparation) was not needed to be performed since we already had the restored files.

9. For setup [Preparation and step 4](https://github.com/slittorin/home-assistant-setup#preparation):
   - At directory `/root`
   - `ssh-copy-id pi@192.168.2.30`
   - `chmod 700 /root/.ssh/*`
   - `cd /config/.ssh`
   - `cp /root/.ssh/* .`

10. At this stage, comment out rows related to Grafana in `/srv/docker-compose.yml`.

11. For [Setup of InfluxDB](https://github.com/slittorin/home-assistant-setup#installation-for-influxdb), perform the following:
    - Since I had not updated my Influx instance, I knew that it was still on version 2.1.1, and we want to restore data to the same version.
    - Step 1 through 3 is not needed since we have restored the files.
    - For step 4: Update `/srv/docker-compose.yml` so that image and `influxdb:latest` is changed to `influxdb:2.2.0-alpine` (note that it is not `influxdb:2.1.1-alpine`, due to the bug I isolated below).
    - Step 5 is not needed since we have restored the files.
    - Perform the activities in step 6.
    - Now we have a container running for Influx, with the right version, and will need to restore database:
      - Since we got a new fresh instance, we also got the bucket HA, that we cannot restore directly to.
      - I logged into Influx `http://192.168.2.30:8086/` and proceeded to `Data` -> `Buckets` -> `ha`, and changed the name of the bucket to `ha1`.
        - Thereafter I would be able to perform a normal restore (not full).
      - Isolate the latest tar-file for backup in `/srv/ha-history-db/backup`.
        - In this case it was: `influxdb-backup-daily-20230103_000101.tar`.
          - I could not trust the files in `/srv/ha-history-db/backup/backup.tmp` as these was created after the SSD-errors occured.
      - Create directory `/srv/ha-history-db/backup/restore`.
      - Extract the tar-file with `sudo tar xvf influxdb-backup-daily-20230103_000101.tar -C ./restore` (in backup directory).
      - Go into the container with `sudo docker-compose exec ha-history-db bash`.
        - We have the backup directory on the host mounted, and we have extracted all files.
          - These now resides in directory `/backup/restore/srv/ha-history-db/backup/backup.tmp` on the container.
        - Run the following command `influx restore -b ha /backup/restore/srv/ha-history-db/backup/backup.tmp`.
          - **NOTE** When I first tried to perform `influx restore -b ha /backup/restore/srv/ha-history-db/backup/backup.tmp` below, I got the error `Error: failed to restore SQL snapshot: InfluxDB OSS-only command failed: 500 Internal Server Error: An internal error has occurred - check server logs`.
            - Through `docker logs ha-history-db` I got more insight into the error `Error while freeing cold shard resources`, and that it was related to a race condition [bug](https://github.com/influxdata/influxdb/commit/39eeb3e), so I changed from 2.1.1-alpine to 2.2.0-alpine above, as the bug is fixed in this version.
          - The output should be like the following:\
            `2023/01/12 17:25:00 INFO: Restoring bucket "2fea3c080297848f" as "ha"`\
            `2023/01/12 17:25:01 INFO: Restoring TSM snapshot for shard 1`\
	    up to\
            `2023/01/12 17:25:49 INFO: Restoring TSM snapshot for shard 51`\
	    With no error at the end.
      - We now have a database restored.
        - **NOTE** We had no errors in restore, where however after a week I saw that data is missing between 8/8-2022 to 15/8-2022.
          - Unknown cause and I will not try to restore this time period.
          - See further in [After SSD disk crash, restore of data from MySQL to InfluxDB - Restore failure](https://github.com/slittorin/home-assistant-maintenance/blob/main/README.md#after-ssd-disk-crash-restore-of-data-from-mysql-to-influxdb---restore-failure)
          - Can this be related to one of the shard not being restored?
    - Perform the activities in step 7.
    - Log into Influx `http://192.168.2.30:8086/` and proceeded to `Data` -> `Buckets` -> `ha`, and remove the bucket `ha1` (renamed above).

12. For [Backup of InfluxDB](https://github.com/slittorin/home-assistant-setup#backup-for-influxdb):
    - Perform step 1 and 2.

13. Remove the commments added in step 9, related to Grafana in `/srv/docker-compose.yml`.

14. For [Installation of Grafana](https://github.com/slittorin/home-assistant-setup#installation-for-grafana):
    - Since I had not updated my Grafana instance, I knew that it was still on version 8.3.3, and we want to restore data to the same version.
    - Step 1 through 3 is not needed since we have restored the files.
    - For step 4: Update `/srv/docker-compose.yml` so that image and `grafana:latest` is changed to `grafana:8.3.3`.
    - Step 5 is not needed since we have restored the files.
    - Perform the activities in step 6.
    - Now we have a container running for Grafana, with the right version, and will need to restore database:
      - Isolate the latest tar-file for backup in `/srv/ha-grafana/backup`.
        - In this case it was: `grafana-backup-daily-20230103_000201.tar`.
          - I could not trust the files in `/srv/ha-grafana/backup/backup.tmp` as these was created after the SSD-errors occured, and the 4/4 file was empty.
      - Create directory `/srv/ha-grafana/backup/restore`.
      - Extract the tar-file with `sudo tar xvf grafana-backup-daily-20230103_000201.tar -C ./restore` (in backup directory).
      - We stop the container to allow us to copy file without a running Grafana with `sudo docker-compose stop ha-grafana`.
      - Copy the database file to the container image with `sudo docker cp /srv/ha-grafana/backup/restore/srv/ha-grafana/backup/backup.tmp/grafana.db ha-grafana:/var/lib/grafana/`
      - Start the container again with `sudo docker-compose up -d`
      - Ensure that the permissions and ownership is correct with `docker exec -it --user root ha-grafana bash` and the following:
        - `cd /var/lib/grafana`
	- `chown grafana grafana.db`
	- `chmod ug+rw grafana.db`
      - Stop the container again with `sudo docker-compose stop ha-grafana`, then start again with `sudo docker-compose up -d`.
    -  Update token in step 7 in Grafana.
    - Login to Grafana with `http://192.168.2.30:3000/` and ensure that all dashboards are there.
      - Since I had to restore a backup that did not contain all changes made, I needed also to restore JSON [manually](https://community.grafana.com/t/how-to-import-a-panel-using-its-json-data/29715).

15. For [Backup of Grafana](https://github.com/slittorin/home-assistant-setup#backup-for-grafana-database):
    - Perform step 2 and 3.

16. For [Setup of Home Assistant](https://github.com/slittorin/home-assistant-setup#setup-for-home-assistant):
    - Perform the steps for [History DB setup](https://github.com/slittorin/home-assistant-setup#history-db-setup).

17. For Setup of [OS/HW statistics](https://github.com/slittorin/home-assistant-setup#oshw-statistics):
    - Perform step 3 and 4 to add to crontab.

18. For Setup of [Docker volume sizes](https://github.com/slittorin/home-assistant-setup/blob/main/README.md#docker-volume-sizes):
    - Perform step 3 and 4 to add to crontab.
 
19. Let the whole instance run for a hours and check
    - That data is coming into InfluxDB, through Grafana and appropriate dashboard for the day.
    - Check HA error logs.

20. Let the whole instance run for a few days and check:
    - On Server1:
      - That backup scripts is running correctly.
      - That os-stats script is running correctly.
    - On HA:
      - That Git-triggers from HA for Grafana works correctly.
      - That `/config/scripts/copy_backup.sh` is running correctly
      - That `/config/scripts/remote_docker_volume_size.sh` is running correctly
      - That `/config/scripts/remote_stats.sh` is running correctly

Well thats all folks. Never forget to thorougly test restore of your data and configuration.

### After SSD-disk crash, restore of data from MySql to InfluxDB

The [SSD-disk crash](https://github.com/slittorin/home-assistant-maintenance/blob/main/README.md#failed-ssd-drive) left a hole in the continous data-feed from HA to InfluxDB.

As we do not have any data to restore from InfluxDB, we need to have another source of data.

The source to use is the HA-database in MySQL, where we have either event/state-data or history-data.
In this case, we need to rely on history-data as the event/state-data is purged after a given period.

The history-data is stored in table `statistics` where the id for the sensor is stored in `statistics_meta`.
We have here data for each hour, with `max`, `min`, `mean`, `sum` and for some sensors `state`.
Thus, we can recreate quite a lot of data.

However, there are some caveats:
- We only have data for the hour (stored  at the beginning of the hour).
- We will have data for each hour, even if there are no state-changes for the sensors that hour, i.e. we cannot tell if there is not state-change for this hour (most likely the values are zero).
- We do not have the exact measures (if it is not a sensor where we can use `max`, `min`, `sum` or `state`).
- We will only have one measure to add, and that is the sensor value. We will not get the other measures associated with the sensor (that is normally added by HA).

#### Steps to prepare data.

This is the steps I utilized to be able to get the data converted from MySQL to InfluxDB (yes, I know there are other means, but here we can visualize and verify the data directly):
1. Extract the table `statistics` and `statistics_meta` into csv-files (in this case with MySQL Workbench).
   - Note that the extract shall be precise, so it do not overlap with datetime of data that exists in InfluxDB.
   - Note also to extract with right character-set, so that unit or sensor is correct.
2. Add the data to an excel-matrix.
3. By formulas link the meta_id to sensor-name, and remove 'sensor.' from the name.
4. Isolate the type of data we want to get, in this case `max`, `min`, `mean` or `state`.
5. Build a csv-import string for the sensor.

For example the following csv-import string: `,,0,2023-01-02T00:00:00Z,2023-01-12T21:32:00.00Z,2023-01-12T20:59:57.00Z,-6.307299999999999,value,%,sensor,electrical_daily_calculated_consumption_index_pct`
Is built by creating the string with the following columns:
- Empty first column (`,,`).
- Table-id. This is an increasing value, for each sensor that is added.
- Start-datetime, fixed for all import-rows.
- End-datetime, fixed for all import-rows.
- From `statistics`: The `datetime` of history-data (here converted to ':59:57:00Z' as I want the insert to be in the end of the hour, similar when existing sensors have state-triggered).
- From `statistics`: The `value`. Based on the type of data (see above).
- From `statistics_meta`: The `unit` of the sensor.
- Fixed string `sensor`.
- From `statistics_meta`: The `sensor name`.

For each new sensor, we also add the following headers:
```
#group,false,false,true,true,false,false,true,true,true,true
#datatype,string,long,dateTime:RFC3339,dateTime:RFC3339,dateTime:RFC3339,double,string,string,string,string
#default,_result,,,,,,,,,
,result,table,_start,_stop,_time,_value,_field,_measurement,domain,entity_id
```

I also marked some sensor to not be imported, this as they have been removed after the SSD-crash (see [configuration.yaml](https://github.com/slittorin/home-assistant-config/blob/master/configuration.yaml).\
Also made sure that sensors that was down-sampled after the SSD-crash, was recreated by using `min`, `max` and so forth to create manual data to be inserted.

And lastly I made a VB-script in the excel-matrix to create full text for csv-import.

#### Steps to import the converted data.

I performed the following steps to import the converted data:
1. Copy the csv-import data to a file on `server1`.
2. Split the data into two files, one with only the first sensor, and the other with the rest of the sensors.
   - This so we can verify first with one import.
3. Isolate a number of sensors to verify once import is done.
4. Log into InfluxDB-web, goto Data explorer and Script editor (with raw data).
5. Verify that there is no data for the specific period, with for instance:
   ```
   from(bucket: "ha")
    |> range(start: 2023-01-02T00:00:00Z, stop: 2023-01-12T21:32:00.00Z)
    |> filter(fn: (r) => r["entity_id"] == "sma_total_yield_hour")
   ```
6. Logged into server1, perform:
   - `sudo docker-compose exec ha-history-db bash`.
   - In docker-instance for InfluxDB:
     - Isolate the right file to import (in my case I have placed it in directory under `/srv/ha-history-db/import`, that is reached as `/import` in the docker-instance).
     - Run `influx write -b ha --format csv --skipRowOnError -f FILETOIMPORT`
       - No error should occur (in my case, no message at all).
7. Verify that the first sensor exists in InfluxDB by running step 5.
8. Perform step 6 again with the second import-file.
9. Verify the sensors isolated in step 3 (except the first one) in step 5, to verify that data was added.

Now we can go to HA and look at various dashboard to isolate that the data has been restored, and that they seem to be correct.

By doing the above I got back 142 sensors with data either correct (max, min, state) or as close as possible (mean).\
This out of 230+ sensors.

#### Steps to export data

Since we did not have any data in InfluxDB for the period 2023-01-02 through 2023-01-12, we did not have have any export data.\
See [export for InfluxDB](https://github.com/slittorin/home-assistant-setup#export-for-influxdb) on how we export.

We need to do this manually with the following on server1:
- `sudo /srv/influxdb-export -f 20230102 -t 20230112`
- Verify in dir `/srv/ha-history-db/export/2023` that the export-files are correctly created.
  - Note that the files will be smaller since we do not have all measures restored (only measure for value).

And finally we have restored history-data that was not in InfluxDB due to the SSD-crash.

Hereafter the backup for InfluxDB will also contain the data from the import.

### After SSD-disk crash, restore of data from MySql to InfluxDB - Restore failure

The restore of data after the [failed SSD drive](https://github.com/slittorin/home-assistant-maintenance/blob/main/README.md#failed-ssd-drive), yielded a 'hole' in the data between 2022-08-07 and 2022-08-15.

After analysis of the data in the history-table in MySQL, I can determine that data already started to decay/become intermittent during 7/8.\
and this occurs during 8/8 through 15/8 to approx. 23.00.\
Here I am not sure what this depends on, possibly this is related to malfunction in HA that was resolved after a reboot.

In InfluxDB, I can see the decay/intermittent data in 7/8 and in 15/8, but no data between 8/8 through 14/8.\
Thus, when performing restore of data in the same manner as in [After SSD disk crash, restore of data from MySQL to InfluxDB](https://github.com/slittorin/home-assistant-maintenance/blob/main/README.md#after-ssd-disk-crash-restore-of-data-from-mysql-to-influxdb), I need to take this into account.

I ended up to only restore data for 8/8 through 14/8, this to not have double-data, or wrong data written on 7/8 and 15/8.\
This means that the data cannot be trusted at 7/8 and 14/8, where-as the data can be considered as near as truth as possible between 8/8 and 14/8.

I followed the instructions in [After SSD disk crash, restore of data from MySQL to InfluxDB](https://github.com/slittorin/home-assistant-maintenance/blob/main/README.md#after-ssd-disk-crash-restore-of-data-from-mysql-to-influxdb), with adaptions for the data.

In total 116 sensors was restored.\
This is vastly lower than 142 sensors for above, and 230+ sensors in total.

### After update and reboot of server1 caused the ha-database to be lost

After login to server1, and performing an update of the OS (incl. docker and docker-compose), and after reboot, the InfluxDB 'ha' bucket was empty, and all API-tokens was lost (Admin token was set to other value).\
No data was shown in Grafana (No data), and HA did not write data to InfluxDB.

InfluxDB was not set into reset mode, for new installation, instead I could login fine with the admin-password and the 'ha' was there.

I could not isolate any errors or warnings with `docker logs ha-history-db`.\
So, a mystery....!

I performed the below steps to get the InfluxDB running again.

#### Update Root (admin)-token

As the root-token was also changed, I needed to change in server1, so that scripts would be able to run.

1. Login to server1 and to InfluxDB.
2. Copy the API-token for admin from InfluxDB into the variable `HA_HISTORY_DB_ROOT_TOKEN` in the file `/srv/.env' on server1.

#### Restore the old InfluxDB-database

We have two options, restore the database, or import all exported daily-files.\
We choose to restore the database.

Note that this will remove/overwrite the 'ha' bucket.

The update on server1 was made roughly 15.00 on 2024-01-09 and the InfluxDB bucket 'ha' was up again around 16.30.\
As we can only restore data from the backup made during the night, we loose a substantial amount of data for the day.

1. Uncompress the latest backup.
   - In directory `/srv/ha-history-db/backup`, uncompress with `tar xvf FILENAME.tar`.
   - The dirs/files are exported in the same directory, so copy them to `/srv/ha-history-db/backup/backup.tmp/`.
2. Change the permission for the files in `/srv/ha-history-db/backup/backup.tmp/` with `chmod ugo+r *`.
3. Login to Login into InfluxDB and remove bucket 'ha'.
4. Login to the container with `sudo docker-compose exec ha-history-db bash`.
   - Start the restore with `influx restore /backup/backup.tmp --bucket ha`.
   - Log files similar to this should occur `2024/01/09 17:00:12 INFO: Restoring TSM snapshot for shard 105`.

#### Recreate API-tokens

1. Login into InfluxDB.
1. I recreated the API-token for 'Write to HA bucket' thas is used in Home Assistant.
   - Added the new token in file `secrets.yaml' in the `config` directory in HA.
   - And thereafter restarted HA.
   - Data was then written to the InfluxDB-database correctly by HA.
2. I recreated the API-token for 'Read from HA bucket' thas is used in Grafana.
   - I added the new token in for data source 'ha_history_db' in Grafana.
   - After this I could get data in Grafana to show up.

#### Verify that backup works

1. Login to server1.
2. In the directory `/srv/` run `sudo ./influxdb-backup.sh`.
3. Verify the log `/srv/log/influxdb-backup.log`.
   - So that no error has occured.
4. Verify that database-file (for example `influxdb-backup-daily-20240109_180550.tar`)created in dir `/srv/ha-history-db/backup` seems correct in size.

After this we should have a functional InfluxDB database/bucket for Home Assistant.
