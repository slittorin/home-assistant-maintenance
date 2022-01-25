# Home Assistant - Maintenance

## Table of content

- [Incorrect SMA Energy data](https://github.com/slittorin/home-assistant-maintenance#incorrect-sma-energy-data)

## Incorrect SMA Energy data

At least one time, the data in Home Assistant for SMA Inverter and Home Manager 2.0 has been wrong.\
In this case, there was a peak at 25/1-2022 where produced (consumed) solar was over 2700 kWh early in the morning.

### This is how I identified and corrected the problem.

#### For Recorder database (MariaDB) i am using MySQL Workbench:

1. Identify the different meta-id that you need to look at through `select * from homeassistant.statistics_meta`.
   - You may need to identify several that applies to the error.
2. For the identified meta-ids, run sql command to look at the data:
   - With for instance `select * from homeassistant.statistics where metadata_id = 4`.
   - In this case it was metadata_id 4 `sensor.total_yield` and 5 `sensor.pv_gen_meter` that contained the error where column `state` was 0 for a number of hours.
3. I updated the column `state` to the previous value, before it was set to zero (between 00:00 and 06:00):
   - For all valid ids identified for the above different metadata_ids.
     - Update with `update homeassistant.statistics set state = 2756.716 where id = 2879` 
   - This was during the night where no solar production was made. So it was easy to correct.
4. I also needed to update the column `sum` to correctly reflect the increase:
   - This was more tricky, as I needed to calculate the increase manually.
   - For all valid ids identified for the above different metadata_ids.
     - Update With `update homeassistant.statistics set sum = 21.105 where id = 3002` for all valid ids identified and updated incremental sum.
5. In this case i also restarted Home Assistant, but I am not sure if this is needed.

#### For the InfluxDB history database, the following was performed:

1. TBD
