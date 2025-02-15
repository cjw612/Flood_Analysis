# Flood Damage Analysis from 2019 to 2023 in Taiwan
Analysis of flood data from 2019 - 2023 in Taiwan

![Flood Map](assets/flood_map.jpg)
*Example flood map of Taiwan*

- ### Project Summary
  - #### Project Overview
    This project aims to estimate the historical flood losses from 2019 to 2023 in various townships and districts across Taiwan using data collected from flood sensors, coupled with administrative boundaries and flood damage functions to provide a theoretically sound estimate of flood damage. As a result, this project aims to construct a comprehensive list of flood events that happened in Taiwan from 2019 to 2023, in addition to a geographical visualization of the estimated damage of the flood events. The datasets used in this project include flood sensors, flood records, shapefiles for administrative boundaries, and economic indices for estimating financial losses. This project integrates data preprocessing, feature transformation, spatial data processing, and dynamic data visualization.
  
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

- ### Data Structure and Data Preprocessing
   - #### Flood Record Data
     Due to the massive amount of raw data, to optimize the reading process, data preprocessing and filtering are performed when reading each entry. Specifically, each entry in the flood record data represents one unique record of one unique flood recording station at a specific time. In addition, not all flood recording stations record in centimeters, some stations record in other units such as dBm or Voltage, which may raise inconsistencies when dealing with mixed units. Therefore, we only record entries that fit the following criteria:
     1. Has a measurement unit of "cm" (centimeters)
     2. Records flood depth instead of other vatiables
     3. Has a record value that is greater than 0
     Entries that fit the aforementioned criteria would be recorded into the dataframe. However, as not all columns provide information relavant for analyzing flood depth and damage, only columns *station_id* (used to join with the primary key of the sensors table) *timestamp* (records the time of the event) 


  | sensorID | timestamp           | floodDepth | district       | economicDamage |
  |----------|---------------------|------------|---------------|----------------|
  | 00123    | 2023-06-15 14:00    | 75 cm      | Taipei City   | NT$ 5,000,000  |
  | 00456    | 2023-07-10 09:30    | 120 cm     | Kaohsiung     | NT$ 10,500,000 |
  | 00890    | 2023-08-21 16:45    | 40 cm      | Taichung      | NT$ 2,750,000  |

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



https://tradingeconomics.com/euro-area/consumer-price-index-cpi (taiwan cpi)

https://www.ris.gov.tw/info-popudata/app/awFastDownload/file/y0s6-00000.xls/y0s6/00000/ (area of each district)

https://cybsbox.cy.gov.tw/CYBSBoxSSL/edoc/download/68467 (10 cm cutoff)

Hypothesis:
1. Only include flood sensors 與縣市政府合建
2. Only include flood sensors that use CM (detects depth)
3. An incident is Define an incident cutoff (e.g., if two consecutive records in the same county/district have a time gap >1 hour, they belong to different incidents).
