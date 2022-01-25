# Home Assistant - Maintenance

## Table of content

- [Incorrect SMA Energy data](https://github.com/slittorin/home-assistant-maintenance#incorrect-sma-energy-data)

## Incorrect SMA Energy data

At least one time, the data in Home Assistant for SMA Inverter and Home Manager 2.0 has been wrong.\
In this case, there was a peak at 25/1-2022 where produced (consumed) solar was over 2700 kWh early in the morning.\
Others have also had this problem, see [here](https://community.home-assistant.io/t/sma-solar-sensor-pv-gen-meter-showing-inconsistent-data/368280) and [here](https://github.com/home-assistant/core/issues/61838).

### This is how I identified and corrected the problem.

#### For Recorder database (MariaDB) i am using MySQL Workbench:

Note that one only has 5 minute window to correct, otherwise `statistics_short_term` table will have new data to correct (and if over each hour, `statistics` table.

Be extra cautious with the sql-update commands, preferably take a backup before updating.

1. Identify the different meta-id that you need to look at through `select * from homeassistant.statistics_meta`.
   - You may need to identify several that applies to the error.
   - For this problem the following was chosen to investigate:
     - metadata_id 4: `sensor.total_yield`
     - metadata_id 5: `sensor.pv_gen_meter`.
     - metadata_id 8: `sensor.metering_total_yield`.
     - metadata_id 9: `sensor.metering_total_absorbed`.
   - Sensor data for these, resides in the following tables:
     - `statistics` for hourly data.
     - `statistics_short_term` for 5 minute data.
2. For the identified meta-ids, run sql command to look at the data:
   - With:
     - `select * from homeassistant.TABLE where metadata_id = METADATA_ID order by created desc;`.
   - For this problem, it occured when `state` was set to 0 (zero).
3. First we need to correct where `state` is 0 (zero) for identified tables and metadata_ids.
   - Do this by updating `state` column to the last valid value (data is not updating during the period).
4. Then we need to correct `sum` column so it is correctly reflects the increase.
   - Do this by updating the `sum` column from the where it is clear that the sum-increase is too large.
5. Remember to verify that no new values has been written to the `statistics` and `statistics_short_term` tables.

To update both `state` and `state` column, I copied the data into a excel-matrix and made sql-commands based on the data.
I did not utilize the python program suggested above.

#### For the InfluxDB history database, the following was performed:

1. TBD
