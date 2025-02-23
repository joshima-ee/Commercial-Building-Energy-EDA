# Commercial Building Energy Exploratory Data Analysis

## Introduction
&nbsp;&nbsp;&nbsp;This exploratory data analysis (EDA) aims to assess the energy efficiency of a zero-energy commercial building in Charlottesville, Virginia, and evaluate whether its 12.3 kWp solar PV system meets its energy demands. The insights will help identify energy-saving opportunities and optimization strategies. The dataset is acquired through Kaggle (https://www.kaggle.com/datasets/claytonmiller/building-energy-use-production-and-air-leakage). This EDA utilizes the following Excel files from the dataset:
- Hourly building energy use
- Monthly energy production
- Average weather temperature

&nbsp;&nbsp;&nbsp;The following points are to be tackled:
1. How is energy consumption distributed per category?
1. What is the annual trend in its energy consumption?
1. Is the building really a zero-energy building?
1. How has the building’s 12.3 kWp solar photovoltaic (PV) system performed over the years? 
1. What is the relationship between energy consumption and average weather temperature?
1. What does a typical day look like?
   
&nbsp;&nbsp;&nbsp;To accomplish this EDA, the following tools will be utilized:
- Microsoft Excel and Powerquery for initial data cleaning and transformation
- SQLite with DBrowser for data cleaning and transformation
- Python with Jupyter Lab for analysis and visualization
- Gemini for coding assistance

## Data Cleaning and Transformation
&nbsp;&nbsp;&nbsp;The first step is data cleaning. SQLite will be the primary tool for data cleaning; however, the dataset comes as Excel files in wide format. As an initial step, Powerquery will be used to transform it to a long format and prepare the files for CSV conversion for easier data manipulation later. 

&nbsp;&nbsp;&nbsp;The first file to be processed is the “Hourly_Whole_Blg_ByCircuit” worksheet in the “Whole Building” workbook with the following columns:
- Date  
- Time  
- 1st flr AHU  
- 1st flr HP  
- 1st flr Lights  
- 1st flr Lobby recp  
- 1st flr Office #1 recp  
- 1st flr Office #2 recp  
- 1st flr Office #3 recp  
- 1st flr Bathroom  
- 1st flr Kitchen  
- 1st flr Copy Room recp  
- 1st flr Utility Room recp  
- 2nd flr AHU - Classroom  
- 2nd flr AHU - Computer Room  
- 2nd flr HP - Classroom  
- 2nd flr HP - Computer Room  
- 2nd flr Office recp  
- 2nd flr Oven  
- 2nd flr Lights  
- 2nd flr Computer Room recp  
- 2nd flr Classroom #1 recp  
- 2nd flr Classroom #2 recp  
- 2nd flr Bathroom  
- 2nd flr Kitchen  
- 2nd flr Kitchen recp + Dishwasher  
- 2nd flr Water Cooler  
- 2nd flr Computer Room + Kitchen recp  
- 2nd flr Classroom #2 + Copy Room recp  
- 2nd flr Storage Room + Computer Room recp  
- Refrigerator  
- Exterior Lights  
- ERV  
- Water Heater  

<p align="center">
  <img src="https://github.com/joshima-ee/Commercial-Building-Energy-EDA/blob/main/Images/Excel%20dataset.png" alt="Excel Dataset">
</p>

&nbsp;&nbsp;&nbsp;In Powerquery, the columns are reduced to 4 columns (Date, Time, Circuit, & EnergyConsumption) by unpivoting the circuits using the following M code:

```
let
    Source = Excel.CurrentWorkbook(){[Name="Table1"]}[Content],
    #"Changed Type" = Table.TransformColumnTypes(Source,{{"Date", type date}, {"Time", type time}, {"1st flr AHU", type number}, {"1st flr HP ", type number}, {"1st flr Lights", Int64.Type}, {"1st flr Lobby recp", Int64.Type}, {"1st flr Office #1 recp", Int64.Type}, {"1st flr Office #2 recp", Int64.Type}, {"1st flr Office #3 recp", Int64.Type}, {"1st flr Bathroom", Int64.Type}, {"1st flr Kitchen", Int64.Type}, {"1st flr Copy Room recp", Int64.Type}, {"1st flr Utility Room recp", Int64.Type}, {"2nd flr AHU - Classroom ", type number}, {"2nd flr AHU - Computer Room", type number}, {"2nd flr HP - Classroom", type number}, {"2nd flr HP - Computer Room", type number}, {"2nd flr Office recp", Int64.Type}, {"2nd flr Oven", type number}, {"2nd flr Lights", Int64.Type}, {"2nd flr Computer Room recp", Int64.Type}, {"2nd flr Classroom #1 recp", Int64.Type}, {"2nd flr Classroom #2 recp", Int64.Type}, {"2nd flr Bathoom", Int64.Type}, {"2nd flr Kitchen", Int64.Type}, {"2nd flr Kitchen recp + Dishwasher", Int64.Type}, {"2nd flr Water Cooler", Int64.Type}, {"2nd flr Computer Room + Kitchen recp", Int64.Type}, {"2nd flr Classroom #2 + Copy Room recp", Int64.Type}, {"2nd flr Storage Room + Computer Room recp", Int64.Type}, {"Refridgerator", Int64.Type}, {"Exterior Lights", Int64.Type}, {"ERV", Int64.Type}, {"Water Heater", type number}}),
    #"Unpivoted Other Columns" = Table.UnpivotOtherColumns(#"Changed Type", {"Date", "Time"}, "Attribute", "Value"),
    #"Renamed Columns" = Table.RenameColumns(#"Unpivoted Other Columns",{{"Attribute", "Circuit"}, {"Value", "EnergyConsumption"}})
in
    #"Renamed Columns"
Next is “Energy Production” workbook with the “Monthly Energy Production Data” worksheet. It is worth noting that there are multiple missing month data due to accidental system shutoffs as per the author.  The file initially is in a wide format with year columns and month rows. Powerquery is used to transform the data into a long format through the following M code:
let
    Source = Excel.CurrentWorkbook(){[Name="Table1"]}[Content],
    #"Changed Type" = Table.TransformColumnTypes(Source,{{"Column1", type text}, {"2014", Int64.Type}, {"2015", Int64.Type}, {"2016", Int64.Type}, {"2017", Int64.Type}, {"2018", Int64.Type}, {"2019", Int64.Type}, {"2020", Int64.Type}}),
    #"Unpivoted Columns" = Table.UnpivotOtherColumns(#"Changed Type", {"Column1"}, "Attribute", "Value"),
    #"Renamed Columns" = Table.RenameColumns(#"Unpivoted Columns",{{"Column1", "Month"}, {"Attribute", "Year"}, {"Value", "EnergyProd"}})
in
    #"Renamed Columns"
```

&nbsp;&nbsp;&nbsp;Next is “Energy Production” workbook with the “Monthly Energy Production Data” worksheet. It is worth noting that there are multiple missing month data due to accidental system shutoffs as per the author.  The file initially is in a wide format with year columns and month rows. Powerquery is used to transform the data into a long format through the following M code:

```
let
    Source = Excel.CurrentWorkbook(){[Name="Table1"]}[Content],
    #"Changed Type" = Table.TransformColumnTypes(Source,{{"Column1", type text}, {"2014", Int64.Type}, {"2015", Int64.Type}, {"2016", Int64.Type}, {"2017", Int64.Type}, {"2018", Int64.Type}, {"2019", Int64.Type}, {"2020", Int64.Type}}),
    #"Unpivoted Columns" = Table.UnpivotOtherColumns(#"Changed Type", {"Column1"}, "Attribute", "Value"),
    #"Renamed Columns" = Table.RenameColumns(#"Unpivoted Columns",{{"Column1", "Month"}, {"Attribute", "Year"}, {"Value", "EnergyProd"}})
in
    #"Renamed Columns"

```
<p align="center">
  <img src="https://github.com/joshima-ee/Commercial-Building-Energy-EDA/blob/main/Images/energy%20prod%20excel.png" alt="Energy Prod Data">
</p>


&nbsp;&nbsp;&nbsp;Finally, we transform the “June 2014-December 2016” worksheet in the “Weather_Monthly_SI” workbook. The given file only has data from June 2014 to December 2016. The initial step is to transform this given data so that only the year, month, and average temperature are retained using the following M code:

```
let
    Source = Excel.Workbook(File.Contents("C:\General\Project\Building Energy\Data set\Weather\Weather Data\Monthly\SI Units\Weather_Monthly_SI.xlsx"), null, true),
    #"June 2014-December 2016_Sheet" = Source{[Item="June 2014-December 2016",Kind="Sheet"]}[Data],
    #"Promoted Headers" = Table.PromoteHeaders(#"June 2014-December 2016_Sheet", [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",{{"Column1", type any}, {"Column2", type text}, {"Temperature (°C)", type any}, {"Column4", type any}, {"Column5", type any}, {"Dew Point (°C)", type any}, {"Column7", type any}, {"Column8", type any}, {"Humidity (%)", type any}, {"Column10", type any}, {"Column11", type any}, {"Wind Speed (m/s)", type any}, {"Column13", type any}, {"Column14", type any}, {"Pressure (kpa)", type any}, {"Column16", type any}, {"Column17", type any}, {"Precipitation (cm)", type any}, {"HDD", type any}, {"CDD", type any}, {"Column21", type any}, {"Column22", type any}, {"Column23", type any}, {"Column24", type any}, {"Column25", type any}, {"Column26", type any}, {"Column27", type any}, {"Column28", type any}, {"Column29", type any}, {"Column30", type any}, {"Column31", type any}, {"Column32", type any}, {"Column33", type any}, {"Column34", type any}, {"Column35", type any}, {"Column36", type any}}),
    #"Promoted Headers1" = Table.PromoteHeaders(#"Changed Type", [PromoteAllScalars=true]),
    #"Changed Type1" = Table.TransformColumnTypes(#"Promoted Headers1",{{"Year", Int64.Type}, {"Month", type text}, {"Max", type number}, {"Avg", type number}, {"Min", type number}, {"Max_1", type number}, {"Avg_2", type number}, {"Min_3", type number}, {"Max_4", Int64.Type}, {"Avg_5", type number}, {"Min_6", Int64.Type}, {"Max_7", type number}, {"Avg_8", type number}, {"Min_9", Int64.Type}, {"Max_10", type number}, {"Avg_11", type number}, {"Min_12", type number}, {"Total", type number}, {"(18°C)", Int64.Type}, {"(18°C)_13", Int64.Type}, {"Column21", type any}, {"Column22", type any}, {"Column23", type any}, {"Column24", type any}, {"Column25", type any}, {"Column26", type any}, {"Column27", type any}, {"Column28", type any}, {"Column29", type any}, {"Column30", type any}, {"Column31", type any}, {"Column32", type any}, {"Column33", type any}, {"Column34", type any}, {"Column35", type any}, {"Column36", type any}}),
    #"Removed Other Columns" = Table.SelectColumns(#"Changed Type1",{"Year", "Month", "Avg"}),
    #"Rounded Off" = Table.TransformColumns(#"Removed Other Columns",{{"Avg", each Number.Round(_, 2), type number}}),
    #"Removed Blank Rows" = Table.SelectRows(#"Rounded Off", each not List.IsEmpty(List.RemoveMatchingItems(Record.FieldValues(_), {"", null}))),
    #"Renamed Columns" = Table.RenameColumns(#"Removed Blank Rows",{{"Avg", "Average Temperature"}})
in
    #"Renamed Columns"
```
<p align="center">
  <img src="https://github.com/joshima-ee/Commercial-Building-Energy-EDA/blob/main/Images/weather%20excel.png" alt="Weather Data">
</p>

&nbsp;&nbsp;&nbsp;Next is to acquire additional weather data from wunderground.com. Manually creating a query for each month will be time consuming, thus Python will be used to automate some of the process later. First, a template must be created to extract the relevant table from the website and transform it to only retain the average temperature. The following M code for the query named “AverageMonthlyTemp” became the template:

```
let
    Source = Web.BrowserContents("https://www.wunderground.com/history/monthly/us/va/charlottesville/KCHO/date/2017-5"),
    #"Extracted Table From Html" = Html.Table(Source, {{"Column1", "DIV.summary-table > TABLE.ng-star-inserted > * > TR > :nth-child(1)"}, {"Column2", "DIV.summary-table > TABLE.ng-star-inserted > * > TR > :nth-child(2)"}, {"Column3", "DIV.summary-table > TABLE.ng-star-inserted > * > TR > :nth-child(3)"}, {"Column4", "DIV.summary-table > TABLE.ng-star-inserted > * > TR > :nth-child(4)"}, {"Column5", "DIV.summary-table > TABLE.ng-star-inserted > * > TR > :nth-child(5)"}, {"Column6", "DIV.summary-table > TABLE.ng-star-inserted > * > TR > :nth-child(6)"}}, [RowSelector="DIV.summary-table > TABLE.ng-star-inserted > * > TR"]),
    #"Changed Type" = Table.TransformColumnTypes(#"Extracted Table From Html",{{"Column1", type text}, {"Column2", type text}, {"Column3", type text}, {"Column4", type text}, {"Column5", type text}, {"Column6", type text}}),
    #"Removed Columns" = Table.RemoveColumns(#"Changed Type",{"Column2", "Column4", "Column5", "Column6"}),
    #"Filtered Rows" = Table.SelectRows(#"Removed Columns", each ([Column1] = "Avg Temperature")),
    #"Transposed Table" = Table.Transpose(#"Filtered Rows"),
    #"Promoted Headers" = Table.PromoteHeaders(#"Transposed Table", [PromoteAllScalars=true]),
    #"Changed Type1" = Table.TransformColumnTypes(#"Promoted Headers",{{"Avg Temperature", type number}})
in
    #"Changed Type1"
```

&nbsp;&nbsp;&nbsp;It can be observed that to get weather data for other months, one can simply modify the date on the latter part of the source link. With the help of Gemini AI, the following Python script is devised to automate the creation of monthly queries:

```python
import win32com.client
import datetime
import uuid

def update_weather_query(excel_file_path, base_query_name, start_year, start_month, end_year, end_month):
    try:
        # Open Excel
        excel = win32com.client.Dispatch("Excel.Application")
        excel.Visible = True  # Keep True for debugging, False for background execution

        wb = excel.Workbooks.Open(excel_file_path)

        # Get existing queries
        existing_queries = {q.Name for q in wb.Queries}
        if base_query_name not in existing_queries:
            print(f"Error: Base query '{base_query_name}' not found.")
            print("Available queries:", existing_queries)
            wb.Close(SaveChanges=False)
            excel.Quit()
            return

        base_query = wb.Queries(base_query_name)
        base_m_code = base_query.Formula  # Extract base M code

        # Date range setup
        current_date = datetime.date(start_year, start_month, 1)
        end_date = datetime.date(end_year, end_month, 1)

        while current_date <= end_date:
            year = current_date.year
            month = current_date.month
            month_str = str(month)
            day_str = "1"

            # Generate a unique query name
            new_query_name = f"Weather_{year}_{month_str}_{uuid.uuid4().hex[:8]}"

            # Prevent duplicate query names
            if new_query_name in existing_queries:
                print(f"Skipping duplicate query '{new_query_name}'")
                current_date = datetime.date(year + 1, 1, 1) if month == 12 else datetime.date(year, month + 1, 1)
                continue  # Skip iteration

            # New URL
            new_url = f"https://www.wunderground.com/history/monthly/us/va/charlottesville/KCHO/date/{year}-{month_str}-{day_str}"

            # Modify M Code
            new_m_code = base_m_code.replace(
                "https://www.wunderground.com/history/monthly/us/va/charlottesville/KCHO/date/2017-8",
                new_url
            )

            try:
                # Use `.Add()` instead of `.Duplicate()`
                wb.Queries.Add(new_query_name, new_m_code)
                existing_queries.add(new_query_name)  # Track created queries
                print(f"✅ Created query: {new_query_name}")
            except Exception as e:
                print(f"❌ Error creating query '{new_query_name}': {e}")
                break  # Stop execution if there's an issue

            # Move to next month
            current_date = datetime.date(year + 1, 1, 1) if month == 12 else datetime.date(year, month + 1, 1)

        # **Force Refresh Power Query Connections**
        for conn in wb.Connections:
            try:
                conn.Refresh()
            except Exception as e:
                print(f"⚠️ Warning: Could not refresh connection '{conn.Name}': {e}")

        # Save and close Excel
        wb.Save()
        wb.Close(SaveChanges=True)
        excel.Quit()
        del excel  # Cleanup

        print(f"✅ Queries successfully created and updated in '{excel_file_path}'")

    except Exception as e:
        print(f"❌ An unexpected error occurred: {e}")
```

&nbsp;&nbsp;&nbsp;All queries are then consolidated and transformed to have year, month, and average temperature columns using a query named “AvgTempConsolidation” with the following M code:

```
let
    // Get all queries in the workbook
    AllQueries = #sections[Section1],  

    // Filter only queries matching "Weather_YYYY_M_*"
    FilteredQueries = List.Select(Record.FieldNames(AllQueries), each Text.StartsWith(_, "Weather_")),

    // Extract Year and Month correctly from the query name
    ExtractedData = List.Transform(FilteredQueries, each [
        Year = Text.Middle(_, 8, 4),  // Extract "YYYY"
        
        // Extract only the month (before the "_")
        MonthNumber = Text.BeforeDelimiter(Text.Middle(_, 13, Text.Length(_) - 13), "_"),

        // Map MonthNumber (1-12) to full month names
        Month = 
            if MonthNumber = "1" then "January" 
            else if MonthNumber = "2" then "February"
            else if MonthNumber = "3" then "March"
            else if MonthNumber = "4" then "April"
            else if MonthNumber = "5" then "May"
            else if MonthNumber = "6" then "June"
            else if MonthNumber = "7" then "July"
            else if MonthNumber = "8" then "August"
            else if MonthNumber = "9" then "September"
            else if MonthNumber = "10" then "October"
            else if MonthNumber = "11" then "November"
            else if MonthNumber = "12" then "December"
            else "Unknown",

        // Get the first average temperature from Column1 of the query
        AvgTemp = Record.Field(AllQueries, _)[Avg Temperature]{0}
    ]),

    // Convert the extracted data into a table and only keep Year, Month, and AvgTemp
    ConsolidatedTable = Table.FromRecords(ExtractedData),
    #"Removed Columns" = Table.RemoveColumns(ConsolidatedTable,{"MonthNumber"}),

    // Rename Columns
    RenamedTable = Table.RenameColumns(#"Removed Columns", {{"Year", "Year"}, {"Month", "Month"}, {"AvgTemp", "Average Temperature"}}),
    #"Appended Query" = Table.Combine({RenamedTable, #"June 2014-December 2016"})
in
    #"Appended Query"
```
<p align="center">
  <img src="https://github.com/joshima-ee/Commercial-Building-Energy-EDA/blob/main/Images/AvgTempConsolidation.png" alt="Consolidated Weather Data">
</p>

&nbsp;&nbsp;&nbsp;All generated tables are then saved as comma-separated values (CSV) files before being imported as tables in a SQLite date base via DB Browser. The files are stored in a data base named “blg_energy” and are renamed as “blg_energy_use”, “energy_production”, and “weather” tables.

<p align="center">
  <img src="https://github.com/joshima-ee/Commercial-Building-Energy-EDA/blob/main/Images/Initial%20sqlite.png" alt="Generated SQLite Tables">
</p>

&nbsp;&nbsp;&nbsp;Each table then underwent basic data quality check.

```sql
--Look for null values
SELECT COUNT(*) AS 'Number of Null' 
FROM blg_energy_use 
WHERE Date IS NULL OR Time IS NULL OR Circuit IS NULL OR EnergyConsumption IS NULL;

SELECT COUNT(*) AS 'Number of Null' 
FROM energy_production 
WHERE Month IS NULL OR Year IS NULL OR EnergyProd IS NULL;

SELECT COUNT(*) AS 'Number of Null' 
FROM weather 
WHERE Month IS NULL OR Year IS NULL OR AverageTemperature IS NULL;

--Look for duplicates
SELECT Date, Time, COUNT(*) 
FROM blg_energy_use 
GROUP BY Date, Time 
HAVING COUNT(*) > 32; --There are 32 distinct circuits

SELECT Month, Year, COUNT(*) 
FROM energy_production 
GROUP BY Month, Year 
HAVING COUNT(*) > 1;

SELECT Month, Year, COUNT(*) 
FROM weather 
GROUP BY Month, Year 
HAVING COUNT(*) > 1;

--Look for typos
SELECT DISTINCT Circuit
FROM blg_energy_use;

SELECT DISTINCT Month
FROM energy_production;

SELECT DISTINCT Year
FROM energy_production;

SELECT DISTINCT Month
FROM weather;

SELECT DISTINCT Year
FROM weather;

--Confirm June 2014 - December 2016 data range 
SELECT DISTINCT substr(Date,4,10)
FROM blg_energy_use;

--Examine first and last day of data range 
SELECT DISTINCT Date 
FROM blg_energy_use
WHERE Date LIKE '%12-2016';

SELECT  DISTINCT Date, Time
FROM blg_energy_use
WHERE Date LIKE '13-12-2016';

SELECT DISTINCT Date 
FROM blg_energy_use
WHERE Date LIKE '%06-2014';

SELECT  DISTINCT Date, Time
FROM blg_energy_use
WHERE Date LIKE '18-06-2014';

--Remove 18-06-2014 due to incomplete day data  
DELETE FROM blg_energy_use
WHERE Date = '18-06-2014';

--Confirm deletion
SELECT DISTINCT Date, Time
FROM blg_energy_use
WHERE Date LIKE '%06-2014' ;
```

&nbsp;&nbsp;&nbsp;In SQLite, tables for visualization and analysis will be created.
|Table Name	| Purpose |
------------|----------
| annual_energy_use | Identify and visualize annual energy consumption breakdown |
| hour_energy_use | Analyze energy consumption trends and typical day hourly energy consumption |
| net_energy | Compare energy consumption and production data and average temperature correlation |

&nbsp;&nbsp;&nbsp;The first table to be processed is the “blg_energy_use” table which contains the hourly energy consumption data by circuit. Currently, there are 32 distinct circuits at the circuit column. To analyze energy consumption, these circuits will be grouped into 3 categories: HVAC, Lighting, and Load. 
| Category	| Description |
|-----------|-------------|
|HVAC	| All Air Handling Units (AHUs), Heat Pumps (HPs), and Energy Recovery Ventilator (ERV) |
| Lighting	| All building lighting |
| Load	| All other loads (computers, ovens, refrigerator, etc.) |

```sql
--Cleaned annual energy use by circuit data 
CREATE TABLE IF NOT EXISTS annual_energy_use AS
WITH cte_annual AS (
    SELECT
        substr(Date, 7, 4) AS Year,
        CASE
            WHEN Circuit LIKE '%AHU%' OR Circuit LIKE '%HP%' OR Circuit LIKE 'ERV' THEN 'HVAC'
            WHEN Circuit LIKE '%Lights%' THEN 'Lighting'
            ELSE 'Load'
        END AS Category,
        SUM(EnergyConsumption)/1000.0 AS EnergyConsumption -- Converts to kW and dividing with decimal value ensures real number results
    FROM blg_energy_use
	WHERE Year <> '2014' --2014 has incomplete data 
    GROUP BY Year, Category
)
SELECT
    Year,
    Category,
    EnergyConsumption,
    ROUND((EnergyConsumption * 100.0 / SUM(EnergyConsumption) OVER (PARTITION BY Year)), 2) AS Percentage
FROM
    cte_annual
ORDER BY
    Year, Category;
```

&nbsp;&nbsp;&nbsp;The table for annual energy use was created from the “blg_energy_use” table. The year 2014 was excluded as it does not have complete data. This step also converts the energy from Watts to kiloWatts. Next is to create a categorized hourly energy consumption table.

```sql
--Create categorized hourly table 
CREATE TABLE IF NOT EXISTS hourly_energy_use AS
SELECT DATE(SUBSTR(Date, 7, 4) || '-' || SUBSTR(Date, 4, 2) || '-' || SUBSTR(Date, 1, 2)) AS Date, Time,
	CASE
        WHEN Circuit LIKE '%AHU%' OR Circuit LIKE '%HP%' OR Circuit LIKE 'ERV' THEN 'HVAC'
        WHEN Circuit LIKE '%Lights%' THEN 'Lighting'
        ELSE 'Load'
    END AS Category, SUM(EnergyConsumption) / 1000.0 AS EnergyConsumption

FROM blg_energy_use
GROUP BY Date, Time, Category
ORDER BY Date;

--Confirm table creation
SELECT *
FROM hourly_energy_use;
```

&nbsp;&nbsp;&nbsp;To create a net energy table, the “hourly_energy_use”, “energy_production”, and “weather” will be combined. 

```sql
--Create table for net energy
CREATE TABLE IF NOT EXISTS net_energy AS
WITH cte_use AS(
	SELECT DATE(Date, 'start of month') AS Month, SUM(EnergyConsumption) AS 'MonthlyEnergyUse'
	FROM hourly_energy_use
	GROUP BY Month
	ORDER BY Date
	),
	cte_prod AS(	
	SELECT DATE(Year || '-' || 
         CASE Month
             WHEN 'January' THEN '01'
             WHEN 'February' THEN '02'
             WHEN 'March' THEN '03'
             WHEN 'April' THEN '04'
             WHEN 'May' THEN '05'
             WHEN 'June' THEN '06'
             WHEN 'July' THEN '07'
             WHEN 'August' THEN '08'
             WHEN 'September' THEN '09'
             WHEN 'October' THEN '10'
             WHEN 'November' THEN '11'
             WHEN 'December' THEN '12'
         END || '-01') AS SOMonth, 
		 EnergyProd
	FROM energy_production
	WHERE SOMonth > '2014-05-31'
	ORDER BY SOMonth
	),
	cte_weather AS (
	SELECT DATE(Year || '-' || 
         CASE Month
             WHEN 'January' THEN '01'
             WHEN 'February' THEN '02'
             WHEN 'March' THEN '03'
             WHEN 'April' THEN '04'
             WHEN 'May' THEN '05'
             WHEN 'June' THEN '06'
             WHEN 'July' THEN '07'
             WHEN 'August' THEN '08'
             WHEN 'September' THEN '09'
             WHEN 'October' THEN '10'
             WHEN 'November' THEN '11'
             WHEN 'December' THEN '12'
         END || '-01') AS SOMonth, 
		 AverageTemperature
	FROM weather
	ORDER BY SOMonth
	)
SELECT cte_weather.SOMonth AS Month, cte_use.MonthlyEnergyUse, cte_prod.EnergyProd, cte_weather.AverageTemperature
FROM cte_weather --Selected to ensure complete Month-Year data from June 2014 to December 2020
LEFT JOIN cte_use ON cte_weather.SOMonth = cte_use.Month
LEFT JOIN cte_prod ON cte_weather.SOMonth = cte_prod.SOMonth
ORDER BY cte_weather.SOMonth;

--Confirm table creation
SELECT *
FROM net_energy;
```

## Analysis and Visualization
&nbsp;&nbsp;&nbsp;The tables are then imported to Python for analysis and visualization. Since SQLite does not support date time data type, the relevant columns will be cast into date time using Python after importing the tables into data frames. Also, “Date” and ”Time” column of “hourly_energy_use” will be combined into a single column.

```python
import sqlite3
import pandas as pd

db_path = "C:/General/Project/Building Energy/Data set/blg_energy.db"

#Create connection with blg_energy.db and transform dates to appropriate data type
try:
    with sqlite3.connect(db_path) as conn:
        #Import annual energy use table
        annual_energy_use = pd.read_sql_query("SELECT * FROM annual_energy_use", conn)
        annual_energy_use['Year'] = annual_energy_use['Year'].astype(int)
        annual_energy_use = annual_energy_use.sort_values('Year', ascending=False)

        #Import and transform hourly energy use table
        hourly_energy_use = pd.read_sql_query("SELECT * FROM hourly_energy_use", conn) 
        hourly_energy_use['Date'] = pd.to_datetime(hourly_energy_use['Date'], format='%Y-%m-%d')
        hourly_energy_use['Time'] = pd.to_datetime(hourly_energy_use['Time'], format='%I:%M:%S %p')
        hourly_energy_use['DateTime'] = hourly_energy_use['Date'] + pd.to_timedelta(hourly_energy_use['Time'].dt.strftime('%H:%M:%S'))
        hourly_energy_use = hourly_energy_use.drop(['Date','Time'], axis=1)
        hourly_energy_use.insert(0, 'DateTime', hourly_energy_use.pop('DateTime'))
        hourly_energy_use = hourly_energy_use.sort_values(by='DateTime')

        #Import and transform net energy table
        net_energy = pd.read_sql_query("SELECT * FROM net_energy", conn)
        net_energy['Month'] = pd.to_datetime(net_energy['Month'], format='%Y-%m-%d')

        #Ensure all tables are properly imported and transformed
        print(f"annual_energy_use".center(80,'-'))
        print(annual_energy_use.info())
        print(f"\n")
        print(annual_energy_use.head(3))
        print(f"\n")
        print(f"hourly_energy_use".center(80,'-'))
        print(hourly_energy_use.info())
        print(f"\n")
        print(hourly_energy_use.head(3))
        print(f"\n")
        print(f"net_energy".center(80,'-'))
        print(net_energy.info())
        print(f"\n")
        print(net_energy.head(3))


except sqlite3.Error as e:
    print(f"An error occured: {e}")

except Exception as e:
    print(f"A general error occured: {e}")
```

&nbsp;&nbsp;&nbsp;Let’s look at the relative energy use by category.

```python
#Compare yearly consumption breakdown by category 
import matplotlib
import matplotlib.pyplot as plt

#Transform to wide format for graphing
annual_energy_use_pivot = annual_energy_use.pivot(index='Year', columns='Category', values='Percentage')

#Manually define Set2 colors from seaborn, seaborn does not support stacked bar graphs
set2_colors = ['#66c2a5', '#fc8d62', '#8da0cb']  

#Set plot parameters through matplotlib
ax = annual_energy_use_pivot.plot(kind='barh', stacked=True, figsize=(10, 6), color=set2_colors)

#Formatting
plt.title('Relative Energy Use by Category ', fontsize=22)
plt.ylabel(None)
plt.yticks(fontsize=14)
plt.legend(title='Category', bbox_to_anchor=(1, 1), loc='upper left', frameon=False, title_fontsize=14, fontsize=12)
for p in ax.patches:
    width, height = p.get_width(), p.get_height()
    x, y = p.get_xy() 
    ax.annotate(f'{width:.2f}%', (x + width/2, y + height/2), ha='center', va='center', color='white', fontsize=14)
ax.grid(False)
plt.box(False)
ax.get_xaxis().set_ticks([])
plt.tight_layout()

plt.show()
```
<p align="center">
  <img src="https://github.com/joshima-ee/Commercial-Building-Energy-EDA/blob/main/Images/Annual%20Energy%20Use%20by%20Category.png" alt="Annual Energy Use by Category">
</p>

&nbsp;&nbsp;&nbsp;From the generated stacked bar chart, lighting consistently has the most energy share. Load is the only category to increase from 2015 to 2016 by 4.69% while both HVAC and Lighting decreased.  Let us have a closer look.

```python
#Compare 2015 and 2016 energy consumption
import seaborn as sns

#Set figure size
plt.figure(figsize=(12, 6))

#Set plot parameters through seaborn
ax = sns.barplot(data=annual_energy_use, x='EnergyConsumption' , y='Year', hue='Category', orient='h', palette='Set2', order=['2016', '2015'])

#Formatting
plt.suptitle('Annual Energy Use Breakdown', fontsize=22, ha='center')
plt.xlabel('Energy Use (kW)', fontsize=12)
plt.ylabel(None) 
plt.legend(title='Category', bbox_to_anchor=(1.05, 1), loc='upper left', fontsize=12, frameon=False)

for p in ax.patches:
    width = p.get_width()
    height = p.get_height()
    x = p.get_x()
    y = p.get_y()
    value = width 
    if value > 0:
        ax.annotate(f'{value:.0f}', (x + width/2, y + height/2), 
                    ha='center', va='center', fontsize=14, color='white')    
plt.box(False)
plt.tight_layout()

plt.show()
```
<p align="center">
  <img src="https://github.com/joshima-ee/Commercial-Building-Energy-EDA/blob/main/Images/Annual%20Energy%20Use%20by%20Category%20(2015%20vs%202016).png" alt="Annual Breakdown">
</p>

&nbsp;&nbsp;&nbsp;In this chart, it can be observed that despite the differences in energy share from the previous graph, all categories have increased energy consumption. 

```python
#Calculate energy consumption difference
print(f"2015 vs. 2016 Energy Consumption".center(80,'-'))

#Create dataframes for each category
hvac_data = annual_energy_use[annual_energy_use['Category'] == 'HVAC']
lighting_data = annual_energy_use[annual_energy_use['Category'] == 'Lighting']
load_data = annual_energy_use[annual_energy_use['Category'] == 'Load']

#Input category data by year
energy_2015 = hvac_data[hvac_data['Year'] == 2015]['EnergyConsumption'].values[0]
energy_2016 = hvac_data[hvac_data['Year'] == 2016]['EnergyConsumption'].values[0]

#Compute for difference
kw_difference = round((energy_2016 - energy_2015),2)
percentage_difference = ( kw_difference/ energy_2015) * 100

print(f"Difference in HVAC energy consumption: {kw_difference}kW or {percentage_difference:.2f}%")

energy_2015 = lighting_data[lighting_data['Year'] == 2015]['EnergyConsumption'].values[0]
energy_2016 = lighting_data[lighting_data['Year'] == 2016]['EnergyConsumption'].values[0]

kw_difference = round((energy_2016 - energy_2015),2)
percentage_difference = ( kw_difference/ energy_2015) * 100

print(f"Difference in lighting energy consumption: {kw_difference}kW or {percentage_difference:.2f}%")

energy_2015 = load_data[load_data['Year'] == 2015]['EnergyConsumption'].values[0]
energy_2016 = load_data[load_data['Year'] == 2016]['EnergyConsumption'].values[0]

kw_difference = round((energy_2016 - energy_2015),2)
percentage_difference = ( kw_difference/ energy_2015) * 100

print(f"Difference in load energy consumption: {kw_difference}kW or {percentage_difference:.2f}%")
```
<p align="center">
  <img src="https://github.com/joshima-ee/Commercial-Building-Energy-EDA/blob/main/Images/Annual%20Energy%20Use%20calculation.png" alt="Annual Energy Use Calculation">
</p>

&nbsp;&nbsp;&nbsp;Load energy consumption has the most significant increase at 1596.20 kW or 34.12% increase compared to the previous year. However, lighting has still the most energy consumption at 7206 kW for the year 2016. It is also worth noting that the author pointed out that a change in lighting control system in December 2014 made some exterior lighting to be consistently powered even in daytime.

&nbsp;&nbsp;&nbsp;Let us compare the building’s energy consumption and production and the average weather data. Unfortunately, due to an accidental system shutoff, energy production data is missing some months. To have a complete annual analysis, we will look at the data from July 2014 to July 2015.

```python
#Weather and energy correlation
import matplotlib.pyplot as plt
import seaborn as sns
import pandas as pd

#Create a filter to get data from July 2014 to July 2015
mask = (net_energy['Month'] > '2014-06-30') & (net_energy['Month'] <= '2015-07-31')

#Create dataframe for filtered data and transform data for plotting
net_energy_y1 = net_energy.loc[mask]
net_energy_y1 = net_energy_y1.melt(id_vars=['Month', 'AverageTemperature'], var_name='Type', value_name='Energy')
net_energy_y1['Month'] = net_energy_y1['Month'].dt.strftime('%Y-%b')

#Set figure size
fig, ax1 = plt.subplots(figsize=(15, 6))

#Share x-axis for bar and point plot
ax2 = ax1.twinx()

#Set bar plot parameters for energy production and consumption
sns.barplot(data=net_energy_y1, x='Month', y='Energy', hue='Type', hue_order=['EnergyProd','MonthlyEnergyUse'], palette='Set2', ax=ax1)

#Set point plot parameters for weather
sns.pointplot(data=net_energy_y1, x='Month', y='AverageTemperature', ax=ax2, 
              color='black', label='Average Temperature', markers="D", 
              linestyles="dashed", linewidth=1.2, markeredgewidth=0.8, markerfacecolor="black", markersize=6)

#Formatting
bars, labels1 = ax1.get_legend_handles_labels()  
points, labels2 = ax2.get_legend_handles_labels() 
new_labels = ['Energy Production', 'Energy Consumption', 'Average Temperature']

ax1.legend(bars + points, new_labels, 
           bbox_to_anchor=(1.06, 1), loc='upper left', frameon=False, 
           title="Legend", fontsize=10, title_fontsize=14)
ax2.legend([],[], frameon=False)

plt.suptitle('Monthly Net Energy and Average Weather Temperature', fontsize=22, x=0.44)
ax1.set_xlabel(None)
ax1.set_ylabel('Energy (kW)', fontsize=14)
ax2.set_ylabel('Average Temperature (°C)', fontsize=14)
plt.tight_layout(rect=[0, 0, 0.9, 1]) 

plt.show()
```
<p align="center">
  <img src="https://github.com/joshima-ee/Commercial-Building-Energy-EDA/blob/main/Images/Monthly%20Net%20Energy%20and%20Average%20Weather%20Temperature.png" alt="Monthly Net Energy and Average Weather Temperature">
</p>

&nbsp;&nbsp;&nbsp;As we can see from the graph, there are 5 months (November to March) where the building was on a net negative. The sharp decline in average temperature at the last month of fall in November increased the energy consumption of the building while the temperature increased from the start of spring in March has reduced the energy consumption. The reduced energy production could also be due to the longer nights during winter, limiting the PV system’s productivity.

```python
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt

# Filter data (create a copy to avoid SettingWithCopyWarning)
mask = (net_energy['Month'] > '2014-06-30') & (net_energy['Month'] <= '2015-07-31')
net_energy_net = net_energy.loc[mask].copy()

# Compute net energy & format month names
net_energy_net = net_energy_net.assign(
    Net=net_energy_net['EnergyProd'] - net_energy_net['MonthlyEnergyUse'],
    Month=net_energy_net['Month'].dt.strftime('%Y-%b')
)

# Color bars based on positive/negative values
colors = ['#66c2a5' if c >= 0 else '#fc8d62' for c in net_energy_net['Net']]

# Plot
plt.figure(figsize=(12, 6))
plt.title('Monthly Net Energy', fontsize=22)

ax = sns.barplot(
    data=net_energy_net, x='Month', y='Net', hue=net_energy_net['Month'],  # Assign hue to fix deprecation
    palette=colors, legend=False  # Hide legend
)

# Labels & Formatting
ax.set_ylabel('Net Energy (kW)', fontsize=12)
ax.set_xlabel(None)
ax.axhline(y=0, linewidth=2, color='black')  # Baseline at 0
plt.xticks(rotation=45)  # Rotate x-axis labels for readability

# Add value labels inside the bars
for bar, value in zip(ax.patches, net_energy_net['Net']):
    height = bar.get_height()
    y_position = height / 2 if height >= 0 else height - (height / 2)  # Center inside the bar
    
    ax.text(
        bar.get_x() + bar.get_width() / 2,  # X position (center of bar)
        y_position,  # Center Y position inside bar
        f"{value:.1f}",  # Format label
        ha='center', va='center', fontsize=10
    )

plt.show()
```
<p align="center">
  <img src="https://github.com/joshima-ee/Commercial-Building-Energy-EDA/blob/main/Images/Monthly%20Net%20Energy.png" alt="Monthly Net Energy">
</p>

&nbsp;&nbsp;&nbsp;To determine if the building is net-zero, let’s compute for the difference between the consumption and production between July 2014 to June 2015.

```python
# Define the date range (July 2014 to June 2015)
start_date = "2014-07-01"
end_date = "2015-06-30"

# Filter the data within the date range
filtered_data = net_energy[(net_energy['Month'] >= start_date) & (net_energy['Month'] <= end_date)]

# Calculate total energy consumption and total energy production
total_consumption = filtered_data['MonthlyEnergyUse'].sum()
total_production = filtered_data['EnergyProd'].sum()
net_energy_comp = total_production - total_consumption

# Print the results
print(f"Total Energy Consumption (July 2014 - June 2015): {total_consumption:.2f} kW")
print(f"Total Energy Production (July 2014 - June 2015): {total_production:.2f} kW")
print(f"Building Net Energy (July 2014 - June 2015): {net_energy_comp:.2f} kW")
```
<p align="center">
	<img src="https://github.com/joshima-ee/Commercial-Building-Energy-EDA/blob/main/Images/Building%20Net%20Energy.png" alt="Net Energy Computation">
</p>

&nbsp;&nbsp;&nbsp;Unfortunately, the building is net negative during the period with 522.03 kW. Despite energy surplus on most months, the energy demand during winter could not be offset. 

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np

# Filter annual data
mask1 = (net_energy['Month'] > '2014-06-30') & (net_energy['Month'] <= '2015-06-30')
mask2 = (net_energy['Month'] > '2018-06-30') & (net_energy['Month'] <= '2019-06-30') 
mask3 = (net_energy['Month'] > '2019-06-30') & (net_energy['Month'] <= '2020-06-30') 

final_mask = mask1 | mask2 | mask3

net_energy_pv = net_energy[final_mask].copy()

net_energy_pv['YearGroup'] = net_energy_pv['Month'].apply(lambda x: f"{x.year - 1}-{x.year}" if x.month < 7 else f"{x.year}-{x.year + 1}")

# Get unique year groups
year_groups = net_energy_pv['YearGroup'].unique()

# Create the figure and axes (3 subplots)
fig, axes = plt.subplots(3, 1, figsize=(10, 12))

# Iterate through the year groups and plot
for i, year_group in enumerate(year_groups):
    year_data = net_energy_pv[net_energy_pv['YearGroup'] == year_group]
    sns.barplot(data=year_data, x=year_data['Month'].dt.strftime('%b'), y='EnergyProd', ax=axes[i], color='#66c2a5')
    year_mean = year_data['EnergyProd'].mean()
    axes[i].axhline(year_mean, xmin=0, xmax=1, color='#fc8d62', linestyle='--', label=f'Mean: {year_mean:.2f} kW') # Add mean line

 # Formatting
    start_year, end_year = year_group.split('-')
    axes[i].set_title(f'July {start_year} to June {end_year}')
    axes[i].set_xlabel(None)
    axes[i].set_ylabel('Energy Production (kW)')
    axes[i].tick_params(axis='x')
    axes[i].legend(loc='lower right')
    axes[i].set_yticks(np.arange(0, 2500, 250))
    
plt.suptitle('Annual PV System Energy Production', fontsize=16)
plt.tight_layout(rect=[0, 0, 1, 0.96])
plt.show()
```
<p align="center">
  <img src="https://github.com/joshima-ee/Commercial-Building-Energy-EDA/blob/main/Images/Annual%20Energy%20Production.png" alt="Annual Energy Production">
</p>

&nbsp;&nbsp;&nbsp;To compare a full 12-month period, a July to June year scheme is used to compensate for the missing data.  Most of the year, the building produces more energy than it is consuming, especially from mid spring April to late summer of August where energy production is generally above average. The building could opt to a net metering program with their distribution utility if they are not yet enrolled. This program allows the end user to sell electricity to the grid during excess production at a reduced market rate. This move could alleviate power cost during the winter season.

```python
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

# Create dataframe for HVAC data
hvac_data = hourly_energy_use[hourly_energy_use['Category'] == 'HVAC'].copy()

# Convert DateTime to YearMonth for grouping
hvac_data['YearMonth'] = hvac_data['DateTime'].dt.to_period('M')

# Aggregate HVAC energy use per month
monthly_hvac_energy = hvac_data.groupby('YearMonth')['EnergyConsumption'].sum().reset_index()

# Prepare net_energy for merging
net_energy['YearMonth'] = net_energy['Month'].dt.to_period('M')  # Convert Month to YearMonth

# Merge datasets on YearMonth
hvac_energy_temp = monthly_hvac_energy.merge(
    net_energy[['YearMonth', 'AverageTemperature']], on='YearMonth', how='left'
)

# Compute the average HVAC energy use
avg_energy = hvac_energy_temp['EnergyConsumption'].mean()

# Plot the scatter plot
plt.figure(figsize=(8, 6))
sns.scatterplot(data=hvac_energy_temp, x='EnergyConsumption', y='AverageTemperature', 
                color='#66c2a5', s=100)

# Add reference line for average energy use
plt.axvline(x=avg_energy, color='#fc8d62', linestyle='--', linewidth=2, label=f'Avg Energy Use: {avg_energy:.2f} kW')

# Formatting
plt.title('Monthly HVAC Energy Consumption vs Average Temperature', fontsize=16)
plt.xlabel('Monthly HVAC Energy Use (kW)', fontsize=12)
plt.ylabel('Average Monthly Temperature (°C)', fontsize=12)
plt.xlim(left=0)
plt.ylim(top=30, bottom=-5)
plt.legend()

plt.show()
```
<p align="center">
  <img src="https://github.com/joshima-ee/Commercial-Building-Energy-EDA/blob/main/Images/Energy%20Consumption%20vs%20Ave%20Temperature.png" alt="HVAC vs Temp">
</p>

&nbsp;&nbsp;&nbsp;In general, an average temperature of below 10 °C or above 24 °C corresponds to above average energy consumption while temperature in between have average to below average energy consumption. The graph suggests that cooling is less energy intensive than heating.

```python
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

# Define seasons
def get_season(month):
    if month in [12, 1, 2]:
        return 'Winter'
    elif month in [3, 4, 5]:
        return 'Spring'
    elif month in [6, 7, 8]:
        return 'Summer'
    else:
        return 'Fall'

# Add a 'Season' column
hourly_energy_use['Season'] = hourly_energy_use['DateTime'].dt.month.apply(get_season)

#Set figure size
plt.figure(figsize=(10, 6))

#Set plot parameters in seaborn
sns.boxplot(data=hourly_energy_use, x='EnergyConsumption', y='Season', hue='Season', palette='Set2', legend=False)

#Fornatting
plt.title('Seasonal Energy Consumption Distribution', fontsize=16)
plt.xlabel('Energy Consumption (kW)', fontsize=12)
plt.ylabel(None)
plt.grid(axis='y', linestyle='--', alpha=0.6)

plt.show()
```
<p align="center">
  <img src="https://github.com/joshima-ee/Commercial-Building-Energy-EDA/blob/main/Images/Seasonal%20Energy%20Consumption%20Distribution.png" alt="Seasonal Energy">
</p>
<p align="center">
  <img src="https://github.com/joshima-ee/Commercial-Building-Energy-EDA/blob/main/Images/Seasonal%20Data%20Table.png" alt="Boxplot Stat">
</p>

&nbsp;&nbsp;&nbsp;The box plots show how the hourly energy consumption is distributed by season. The least energy consumption happens during the fall season, maybe due to the comfortable weather at the time with a mean of 0.6 kW and Q3 or 75% of the data on or below 0.73 kW. As expected, winter has the most energy consumption with a mean of 0.91 kW and Q3 of 1.18 kW. Aside from additional heating requirement, lighting load would also increase due to longer nighttime.

```python
# Create columns for Year and Month
hourly_energy_use['Year'] = hourly_energy_use['DateTime'].dt.year
hourly_energy_use['Month'] = hourly_energy_use['DateTime'].dt.month

# Group by Year, Month, and Category
monthly_data = hourly_energy_use.groupby(['Year', 'Month', 'Category'])['EnergyConsumption'].sum().reset_index()

# Define full Year-Month range
years = monthly_data['Year'].unique()
full_index = pd.MultiIndex.from_product([years, range(1, 13)], names=['Year', 'Month'])

# Pivot and reindex to fill missing months with 0
pivot_data = monthly_data.pivot(index=['Year', 'Month'], columns='Category', values='EnergyConsumption').fillna(0)
pivot_data = pivot_data.reindex(full_index, fill_value=0)

#Set number of subplot
num_years = len(years)
fig, axes = plt.subplots(num_years, 1, figsize=(15, 4 * num_years))

# Define category colors
category_colors = {'HVAC': '#66c2a5', 'Lighting': '#fc8d62', 'Load': '#8da0cb'}

# Month labels mapping
month_labels = {1: 'Jan', 2: 'Feb', 3: 'Mar', 4: 'Apr', 5: 'May', 6: 'Jun',
                7: 'Jul', 8: 'Aug', 9: 'Sep', 10: 'Oct', 11: 'Nov', 12: 'Dec'}

# Iterate through years and create stacked bar plots
for i, year in enumerate(years):
    year_data = pivot_data.loc[year]

    # Convert month numbers to names
    month_names = [month_labels[m] for m in year_data.index]

    # Plot
    year_data.plot(kind='bar', stacked=True, ax=axes[i], color=[category_colors[col] for col in year_data.columns], legend=False)

# Formatting
    axes[i].set_title(f'Monthly Energy Consumption - {year}')
    axes[i].set_ylabel('Energy Consumption (kW)')
    axes[i].grid(axis='y', linestyle='--', alpha=0.5)
    axes[i].set_xlabel(None)
    axes[i].set_yticks(np.arange(0, 2750, 250))
    axes[i].set_xticks(range(12))
    axes[i].set_xticklabels(month_names, rotation=0)

plt.setp(axes, xticklabels=month_names)
handles, labels = axes[0].get_legend_handles_labels()
fig.legend(handles, labels, title='Category', loc='upper left', bbox_to_anchor=(0.86, 0.95), fontsize=14, frameon=False, title_fontsize=14)
plt.suptitle('Monthly Energy Consumption by Year', fontsize=16)
plt.tight_layout(rect=[0, 0, 0.85, 1.0])  # Adjust layout to fit legend

plt.show()
```
<p align="center">
  <img src="https://github.com/joshima-ee/Commercial-Building-Energy-EDA/blob/main/Images/Annual%20Energy%20Consumption.png" alt="Categorized Annual Energy">
</p>

&nbsp;&nbsp;&nbsp;Out of the available energy consumption data, January 2016 has most energy consumption. During winter season, HVAC takes up most of the building’s power consumption, while lighting takes up during the other seasons.

```python
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

# Filter for January 14, 2016
jan_data = hourly_energy_use[hourly_energy_use['DateTime'].dt.strftime('%Y-%m') == '2016-01']

# Extract hour for x-axis
jan_data = jan_data.copy()  # Avoid SettingWithCopyWarning
jan_data['Day'] = jan_data['DateTime'].dt.day

# Plot hourly energy consumption
plt.figure(figsize=(12, 6))
sns.barplot(data=jan_data, x='Day', y='EnergyConsumption', errorbar=None, color='#fc8d62')

# Formatting
plt.title('Daily Energy Consumption on January 2016', fontsize=14)
plt.xlabel('Day', fontsize=12)
plt.ylabel('Energy Consumption (kW)', fontsize=12)
plt.xticks(range(0, 31))  # Ensure x-axis shows all hours
plt.show()
```
<p align="center">
  <img src="https://github.com/joshima-ee/Commercial-Building-Energy-EDA/blob/main/Images/January%202016%20Energy%20Consumption.png" alt="January 2016">
</p>

&nbsp;&nbsp;&nbsp;January 19, 2016, is the busiest day of the month, let’s see how energy consumption is distributed throughout that day.

```python
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

# Filter for January 19, 2016
jan_19_data = hourly_energy_use[hourly_energy_use['DateTime'].dt.strftime('%Y-%m-%d') == '2016-01-19']

# Extract hour for x-axis
jan_19_data = jan_19_data.copy()  # Avoid SettingWithCopyWarning
jan_19_data['Hour'] = jan_19_data['DateTime'].dt.hour

# Plot hourly energy consumption
plt.figure(figsize=(10, 6))
sns.lineplot(data=jan_19_data, x='Hour', y='EnergyConsumption', marker='o', linestyle='-', hue='Category', palette='Set2', errorbar=None)

# Formatting
plt.title('Hourly Energy Consumption on January 19, 2016', fontsize=14)
plt.xlabel('Hour of the Day', fontsize=12)
plt.ylabel('Energy Consumption (kW)', fontsize=12)
plt.xticks(range(0, 24))  # Ensure x-axis shows all hours
plt.ylim(top=7, bottom=0)
plt.grid(True, linestyle='--', alpha=0.6)

plt.show()
```
<p align="center">
  <img src="https://github.com/joshima-ee/Commercial-Building-Energy-EDA/blob/main/Images/January%20Hourly%20Energy%20Consumption.png" alt="January Hourly">
</p>

&nbsp;&nbsp;&nbsp;From the given graph, it can be concluded that HVAC is the primary category for energy consumption. HVAC peaked at 7 AM, likely due to morning startup. The HVAC energy consumption from 7 AM onwards is very jagged, which could indicate inefficient operation. Lighting and load pattern indicates most occupants stay between 9 AM to 6 PM. 

&nbsp;&nbsp;&nbsp;Let’s look at April 2016 to compare load distribution on a month with little HVAC usage.

```python
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

# Filter for January 14, 2016
apr_data = hourly_energy_use[hourly_energy_use['DateTime'].dt.strftime('%Y-%m') == '2016-04']

# Extract hour for x-axis
apr_data = apr_data.copy()  # Avoid SettingWithCopyWarning
apr_data['Day'] = apr_data['DateTime'].dt.day

# Plot hourly energy consumption
plt.figure(figsize=(12, 6))
sns.barplot(data=apr_data, x='Day', y='EnergyConsumption', errorbar=None, color='#fc8d62')

# Formatting
plt.title('Daily Energy Consumption on April 2016', fontsize=14)
plt.xlabel('Day', fontsize=12)
plt.ylabel('Energy Consumption (kW)', fontsize=12)
plt.xticks(range(0, 30))  # Ensure x-axis shows all hours

plt.show()
```
<p align="center">
  <img src="https://github.com/joshima-ee/Commercial-Building-Energy-EDA/blob/main/Images/April%202016%20Energy%20Consumption.png" alt="April 2016">
</p>

&nbsp;&nbsp;&nbsp;April 21, 2016, is the busiest day of the month, let’s see how energy consumption is distributed throughout that day.

```python
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

# Filter for January 14, 2016
apr_21_data = hourly_energy_use[hourly_energy_use['DateTime'].dt.strftime('%Y-%m-%d') == '2016-04-21']

# Extract hour for x-axis
apr_21_data = apr_21_data.copy()  # Avoid SettingWithCopyWarning
apr_21_data['Hour'] = apr_21_data['DateTime'].dt.hour

category_order = ['HVAC', 'Lighting', 'Load']

# Plot hourly energy consumption
plt.figure(figsize=(10, 6))
sns.lineplot(data=apr_21_data, x='Hour', y='EnergyConsumption', marker='o', linestyle='-', hue='Category',hue_order=category_order, palette='Set2', errorbar=None)

# Formatting
plt.title('Hourly Energy Consumption on April 21, 2016', fontsize=14)
plt.xlabel('Hour of the Day', fontsize=12)
plt.ylabel('Energy Consumption (kW)', fontsize=12)
plt.xticks(range(0, 24))  # Ensure x-axis shows all hours
plt.ylim(top=7, bottom=0)
plt.grid(True, linestyle='--', alpha=0.6)

plt.show()
```
<p align="center">
  <img src="https://github.com/joshima-ee/Commercial-Building-Energy-EDA/blob/main/Images/April%20Hourly%20Energy%20Consumption.png" alt="April Hourly">
</p>

&nbsp;&nbsp;&nbsp;As expected, HVAC energy consumption is very low and smoother with a peak at 3 PM. Interestingly, lighting load seemed to increase while general load decreased. Lighting and load pattern indicates most occupants stay between 8 AM to 6 PM.

## Conclusion and Recommendations
&nbsp;&nbsp;&nbsp;In an annual basis, the lighting load consumes the most energy per year. The general load of the building has the largest growth out of the 3 categories from 2015 to 2016. The building primarily gets its energy needs through its 12.3 kWp PV solar system except for the winter season where it fails to keep up with the high energy demand from HVAC heating and produce low energy output due to decreased sunlight. The PV solar system has a surplus output for half of the year, however, this could not offset winter energy demand and is a net negative based on the July 2014 – June 2015 analysis.

&nbsp;&nbsp;&nbsp;Seasons have a big effect on energy consumption. From the data set, it can be concluded that an average temperature from 10 °C to 24 °C is the ideal weather temperature to minimize HVAC energy consumption. During an ideal weather, lighting load becomes the major energy consumption. Heating has a higher energy demand compared to cooling. 

&nbsp;&nbsp;&nbsp;With the findings, the following actions are recommended:
- Modify the new lighting control system to allow for turning off exterior lights during the day.
- Utilize dynamic dimmable lighting to minimize daytime lighting load.
- Install motion sensors for lighting to automatically shut off lights in unoccupied areas.
- Review the HVAC system and consider upgrades to increase efficiency in winter.
- Enroll in net metering to help offset the cost of winter power bills.
