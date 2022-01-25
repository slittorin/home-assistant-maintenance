# Home Assistant - Maintenance

## Table of content

- [Incorrect SMA Energy data](https://github.com/slittorin/home-assistant-maintenance#incorrect-sma-energy-data)

## Incorrect SMA Energy data

At least one time, the data in Home Assistant for SMA Inverter and Home Manager 2.0 has been wrong.
In this case, there was a peak at 25/1-2022 where consumed solar was over 2700 kWh early in the morning.

#### This is how I identified and corrected the problem.

I am using MySQL Workbench to connect to MariaDB.

1. Identify the different meta-id that you need to look at through `select * from homeassistant.statistics_meta`.
   - You may need to identify several that applies to the error.
2. For the identified meta-ids, run sql command to look at the data, for instance `select * from homeassistant.statistics where metadata_id = 4`.
   - In this case it was meta-id 4 `sensor.total_yield` that contained the error where column `state` was 0 for a number of hours.
3. I updated the column `state` to the previous value, before it was set to zero.
   - This was during the night where no solar production was made. So it was easy to correct.
