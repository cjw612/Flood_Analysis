# Flood Damage Analysis from 2019 to 2023 in Taiwan
Analysis of flood data from 2019 - 2023 in Taiwan

![Flood Map](assets/flood_map.jpg)
*Example flood map of Taiwan*

- ### Project Summary
  - #### Project Overview
    This project estimates the historical flood losses from 2019 to 2023 in various townships and districts across Taiwan using data collected from flood sensors, coupled with administrative boundaries and flood damage functions to provide a theoretically sound estimate of flood damage. The datasets used in this project include flood sensors, flood records, shapefiles for administrative boundaries, areas for each district, and economic indices for estimating financial losses. This project integrates data preprocessing, feature transformation, spatial data processing, and dynamic data visualization.
    
  - #### Project Objective
    This project aims to construct a comprehensive list of flood events that happened in Taiwan from 2019 to 2023, their corresponding economic damage, in addition to a geographical visualization of the estimated damage of the flood events. 
  
  - #### Tech Stack
    - **Programming Languages:** Python
    - **Libraries & Frameworks:** Pandas, NumPy, Geopandas, Matplotlib
    - **Visualization:** Matplotlib, Tableau
    - **Development & Tools:** Jupyter Notebook, GIS tools

- ### Data Source and Data Preprocessing
  The dataset used for this analysis is collected from various sources:
  - #### Flood Sensor and Flood Record Data:
    Flood sensor and record data are downloaded from the [Civil IoT network](https://history.colife.org.tw/#/). 
  - #### Administrative Boundaries and Geographical Information:
    The SHP file used as the basis for spatial joining flood and geographical data is downloaded from the [National Land Surveying and Mapping Center](https://maps.nlsc.gov.tw/MbIndex_qryPage.action?fun=8). The SHP file used in this project corresponds to the TWD97 EPSG:3824 SHP file. 
  - #### Flood Damage Functions:
    The methodology used in flood damage calculation is based on the methodology suggested in the [European Commission Joint Research Centre (2017)](https://publications.jrc.ec.europa.eu/repository/handle/JRC105688)
  - #### Economic Data:
    Due to flood damage calculation based on the value of Euros in 2010, to convert it to the value of New Taiwan Dollars (NTD) in 2025, [Eurozone CPI in 2010, 2025](https://tradingeconomics.com/euro-area/consumer-price-index-cpi), and the exchange rate of Euros to NTD when the report is written (34.01) is required to perform the conversion.
  - #### District Area Data
    Flood damage calculation is based on the area of each flood event, therefore district-level area information is required to correctly derive the flood damage in each district. The data is downloaded from [The Department of Household Registration](https://www.ris.gov.tw/info-popudata/app/awFastDownload/file/y0s6-00000.xls/y0s6/00000/). 

- ### Data Structure and Data Preprocessing
   - #### Flood Record Data: One row per one unique record of one unique station at a unique time
     Due to the massive amount of raw data, to optimize the reading process, data preprocessing and filtering are performed when reading each entry. Specifically, each entry in the flood record data represents one unique record of one unique flood recording station at a specific time. In addition, not all flood recording stations record in centimeters, some stations record in other units such as dBm or Voltage, which may raise inconsistencies when dealing with mixed units. Therefore, we only record entries that fit the following criteria:
     1. Has a measurement unit of "cm" (centimeters)
     2. Records flood depth instead of other variables
     3. Has a record value that is greater than 0
        
     Entries that fit the aforementioned criteria would be recorded into the dataframe. However, as not all columns provide information relavant for analyzing flood depth and damage, only columns *station_id* (used to join with the primary key of the sensors table) *timestamp* (records the time of the event), and *value* (the value of each observation, which, after filtering out unused data, would be in centimeters) were fetched from the original dataset. The following table is a snapshot of the first five entries of the flood record data, with a total of 10,721,807 entries after dropping duplicates:
  
      | station_id                               | timestamp                  | value    |
      |------------------------------------------|----------------------------|---------:|
      | 38505796-1525-4c8b-9d5c-27fea47db00f     | 2022-07-21 00:00:31.039    | 0.019717 |
      | 38505796-1525-4c8b-9d5c-27fea47db00f     | 2022-07-21 00:09:31.974    | 0.020480 |
      | 38505796-1525-4c8b-9d5c-27fea47db00f     | 2022-07-21 00:11:01.615    | 0.020215 |
      | 38505796-1525-4c8b-9d5c-27fea47db00f     | 2022-07-21 00:19:31.676    | 0.019876 |
      | 38505796-1525-4c8b-9d5c-27fea47db00f     | 2022-07-21 00:21:01.238    | 0.018207 |
     
     *Sample snapshot of flood record dataset*

   - #### Flood Sensors Data: One row per one unique station
     Similarly, only stations that record flood depth in centimeters are recorded into the dataframe. Additionally, only columns *station_id*, *Longitude*, *Latitude*,	and *SIUnit* are read into the dataframe. 18 entries that have abnormal longitude and latitude values are removed after being identified via human inspection of the data. The following table is a snapshot of the first five entries of the flood sensor data, with a total of 1,965 entries after dropping duplicates:

      | station_id                               | Longitude  | Latitude   | SIUnit |
      |------------------------------------------|-----------|-----------|--------|
      | 648c0721-9ae3-4a3b-9007-31dd06a5f293     | 120.241250 | 23.450130 | cm     |
      | b320d298-d3aa-4954-874a-79696f550efa     | 120.188995 | 23.428696 | cm     |
      | c7c0c173-be6c-4fd2-b743-921d987e7330     | 120.392550 | 23.483112 | cm     |
      | bc5af470-def9-4712-95da-8cc29c35fd60     | 120.160995 | 23.508854 | cm     |
      | 54c2b021-edc6-418f-bff5-ec96067b24e6     | 120.433920 | 23.441912 | cm     |

     *Sample snapshot of flood sensor dataset*

  - #### Geographical Data (SHP file): One row per one unique district (village level)
     
- ### Data Cleaning and Preprocessing
  The dataset is cleaned and preprocessed using the following steps:
  - **Feature Engineering:**
    - Derived flood depth differences between sensors and computed overall flood intensity for administrative areas.
    - Spatially joined sensor data with administrative boundaries to categorize events by district.
  - **Variable Transformation:**
    - Transformed categorical indicators such as flood occurrence and administrative zones into binary or categorical variables.
  - **Outlier Removal:**
    - Removed extreme values based on domain knowledge, specifically cases where reported flood depth exceeded realistic values.

- ### Mathematical Model
  The flood loss estimation is based on the following equation:

  \[
  \text{Estimated Damage} = D(m) \times M \times A
  \]

  Where:
  - \( D(m) \) is the damage function dependent on flood depth \( m \)
  - \( M \) is the economic value per unit area
  - \( A \) is the affected area

  The damage function \( D(m) \) is derived from European Commission data and is calculated using an interpolation of provided data points.

  Additionally, outlier filtering was based on the following condition:
  
  \[
  (\text{goldDiff} \geq 4000 \land \text{blueWin} = 0) \lor (\text{goldDiff} \leq -4000 \land \text{blueWin} = 1)
  \]
  
  where goldDiff represents the gold difference between teams, and blueWin indicates if the blue team won.

- ### Exploratory Data Analysis
  The EDA aims to address key questions:
  - **What are the correlations between variables?**
    - Constructed a correlation matrix to assess relationships between flood depth, frequency, and economic loss.
    - Identified unexpected correlations, such as deep floods not always leading to high damage estimates.
  - **What is the distribution of flood intensities?**
    - Plotted histograms of flood depth distributions across different townships.
  - **What are the economic implications of floods?**
    - Analyzed regional variations in economic damages and identified high-risk districts.

- ### Results
  The key findings are summarized below:
  
  | District      | Total Flood Loss (NT$) | Major Flood Events |
  |--------------|----------------------|----------------|
  | Taipei City  | NT$ 1,200,000,000    | 5              |
  | Kaohsiung    | NT$ 800,000,000      | 4              |
  | Taichung     | NT$ 650,000,000      | 3              |
  
- ### Limitations
  - **Sensor Coverage Bias:** Some regions lack adequate sensor installations, leading to potential underestimations.
  - **Simplified Damage Estimation:** The model assumes uniform damage intensity per region, which may not reflect real-world variations.
  - **Exclusion of Other Risk Factors:** Factors such as land use and population density are not fully incorporated.

- ### References
  [1] Water Resources Agency, Taiwan. "Flood Sensor Data." Accessed 2025.  
  [2] European Commission Joint Research Centre. "Global Flood Depth-Damage Functions." 2017.  
  [3] National Land Surveying and Mapping Center, Taiwan. "Administrative Boundary Shapefiles." Accessed 2025.  




https://cybsbox.cy.gov.tw/CYBSBoxSSL/edoc/download/68467 (10 cm cutoff)

Hypothesis:
1. Only include flood sensors 與縣市政府合建
2. Only include flood sensors that use CM (detects depth)
3. An incident is Define an incident cutoff (e.g., if two consecutive records in the same county/district have a time gap >1 hour, they belong to different incidents).
