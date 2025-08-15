Smart Grid — Power BI README (visuals you’re using)

This README documents the exact visuals + measures behind your current report so you (or anyone else) can rebuild/maintain it fast.

1) Data & Setup

Source: smart_grid_curated.csv
Key columns: timestamp (set Data type = Date/Time), meter_id, energy_kwh, real_power_watts, voltage_v, power_factor, high_demand_flag (0/1), anomaly_flag (“Normal”/“Anomaly”), time_of_day.
File → Options → Current File → Data Load: consider disabling Auto date/time.
Data source note - This report was originally connected to Azure Data Lake Storage Gen2 (<account>/<container>/curated) via the native ADLS Gen2 connector with Azure AD authentication.
For reusability and cost control in non-production contexts (sharing, demos, and offline development), the pipeline now exports the curated table to a versioned CSV artifact (smart_grid_curated.csv), and the Power BI query’s Source step points to that file.
The data schema and DAX measures are unchanged; only the connection method was updated. The model can be switched back to ADLS Gen2 at any time by replacing the Source step in Power Query with the ADLS connector.



2) Measures (no Date column required)
-- KPI fundamentals
Total Energy (kWh) = SUM('smart_grid_curated'[energy_kwh])
Avg Voltage (V)    = ROUND(AVERAGE('smart_grid_curated'[voltage_v]), 2)
Global Average Watts = AVERAGE('smart_grid_curated'[real_power_watts])

High Demand % :=
DIVIDE(
    CALCULATE(COUNTROWS('smart_grid_curated'), 'smart_grid_curated'[high_demand_flag] = 1),
    COUNTROWS(ALLSELECTED('smart_grid_curated'))
)

Anomaly % :=
DIVIDE(
    CALCULATE(COUNTROWS('smart_grid_curated'), 'smart_grid_curated'[anomaly_flag] = "Anomaly"),
    COUNTROWS(ALLSELECTED('smart_grid_curated'))
)

Active Meters = DISTINCTCOUNT('smart_grid_curated'[meter_id])

Selected Meter :=
IF(HASONEVALUE('smart_grid_curated'[meter_id]),
   SELECTEDVALUE('smart_grid_curated'[meter_id]),
   "All Meters")

-- Daily energy (built on-the-fly from timestamp)
Avg Daily Energy (kWh) :=
VAR Daily =
    SUMMARIZE(
        ADDCOLUMNS(ALLSELECTED('smart_grid_curated'),
            "__Date", DATEVALUE('smart_grid_curated'[timestamp])),
        [__Date], "kWh", SUM('smart_grid_curated'[energy_kwh]))
RETURN AVERAGEX(Daily, [kWh])

Min Daily Energy (kWh) :=
VAR Daily =
    SUMMARIZE(ADDCOLUMNS(ALLSELECTED('smart_grid_curated'),
        "__Date", DATEVALUE('smart_grid_curated'[timestamp])),
        [__Date], "kWh", SUM('smart_grid_curated'[energy_kwh]))
RETURN MINX(Daily, [kWh])

Max Daily Energy (kWh) :=
VAR Daily =
    SUMMARIZE(ADDCOLUMNS(ALLSELECTED('smart_grid_curated'),
        "__Date", DATEVALUE('smart_grid_curated'[timestamp])),
        [__Date], "kWh", SUM('smart_grid_curated'[energy_kwh]))
RETURN MAXX(Daily, [kWh])

-- Gauge target: average of last 7 days in the current selection (robust)
Target Daily Energy (kWh) :=
VAR Daily =
    SUMMARIZE(ADDCOLUMNS(ALLSELECTED('smart_grid_curated'),
        "__Date", DATEVALUE('smart_grid_curated'[timestamp])),
        [__Date], "kWh", SUM('smart_grid_curated'[energy_kwh]))
VAR Days  = COUNTROWS(Daily)
VAR LastN = TOPN(MIN(7, Days), Daily, [__Date], DESC)
RETURN COALESCE(AVERAGEX(LastN, [kWh]), 0)






3) Visuals
A) Global Average Watts by Time (Line)

X: timestamp → Type: Continuous

Y: [Global Average Watts]

Why: clean trend with spikes/dips.

Tips: Show units in K; add gridlines; enable tooltips (e.g., voltage, power factor).

B) Total Energy (kWh) by Meter ID & Time of Day (Clustered Column with Legend)

Axis: meter_id (Sort by [Total Energy (kWh)] descending)

Legend: time_of_day (Morning/Day/Evening/Night)

Values: [Total Energy (kWh)]

Why: shows when each meter draws energy across the day.

Tip: apply Top N (e.g., 12) filter to keep it readable; data labels on.

C) Average Daily Energy (kWh) by Meter ID (Bar)

Axis: meter_id (sort descending)

Values: [Avg Daily Energy (kWh)]

Why: fair, calendar-normalized comparison of meters.

Tip: rotate axis labels ~45°, keep Top N 20.

D) Target Daily Energy (kWh) Gauge

Value: [Avg Daily Energy (kWh)]

Min / Max: [Min Daily Energy (kWh)] / [Max Daily Energy (kWh)]

Target: [Target Daily Energy (kWh)] (or P90 Daily Energy (kWh))

Why: at-a-glance “are we near target?”





4) KPI Cards (right rail)

Global Average (W): [Global Average Watts]

Meter ID: [Selected Meter] (shows one or “All Meters”)

Total Energy (kWh): [Total Energy (kWh)]

Avg Daily Energy (kWh): [Avg Daily Energy (kWh)]

High Demand (%): [High Demand %] (format as %)

Anomaly (%): [Anomaly %] (format as %)

Avg Voltage (V): [Avg Voltage (V)] (2 decimals; optional conditional color)




5) Interactivity & Formatting

Add slicers for Meter, Anomaly flag, and Date range (on timestamp).

Time-series X-axes → Continuous; display units Thousands (K).

Limit labels to keep visuals clean; rely on tooltips for detail.

Keep the dark theme; use consistent accent colors across pages.





6) Quick sanity checks

If any measure errors on the Gauge Target, confirm timestamp is Date/Time and all four gauge fields use measures (Decimal Number).

If High Demand % looks too high/low, verify high_demand_flag is 0/1 (numeric) and not text.













