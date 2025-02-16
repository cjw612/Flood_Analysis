# 2019 to 2023 Taiwan Flood Damage Estimation
Analysis of flood data from 2019 - 2023 in Taiwan

![Flood Map](assets/flood.png)
*Snapshot of resulting dashboard*

- ### Project Summary
  - #### Project Overview
    This project estimates the historical flood losses from 2019 to 2023 in various townships and districts across Taiwan using data collected from flood sensors, coupled with administrative boundaries and flood damage functions to provide a theoretically sound estimate of flood damage. The datasets used in this project include flood sensors, flood records, shapefiles for administrative boundaries, areas for each district, and economic indices for estimating financial losses. This project integrates data preprocessing, feature transformation, spatial data processing, and dynamic data visualization.
    
  - #### Project Objective
    This project aims to construct a comprehensive list of flood events that happened in Taiwan from 2019 to 2023, their corresponding economic damage, in addition to a geographical visualization of the estimated damage of the flood events. 
  
  - #### Tech Stack
    - **Programming Languages:** Python
    - **Libraries & Frameworks:** Pandas, NumPy, Geopandas, Matplotlib
    - **Visualization:** Matplotlib, Tableau
    - **Development & Tools:** Jupyter Notebook

- ### Data Source and Data Preprocessing
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

- ### Data Structure, Preprocessing, and Schema
   - #### Flood Record Data: One row per one unique record of one unique station at a unique time
     Due to the massive amount of raw data, to optimize the reading process, data preprocessing and filtering are performed when reading each entry. Specifically, each entry in the flood record data represents one unique record of one unique flood recording station at a specific time. In addition, not all flood recording stations record in centimeters, some stations record data in other units such as dBm or Voltage, which may raise inconsistencies when dealing with mixed units. Therefore, we only record entries that fit the following criteria:
     1. Has a measurement unit of centimeters
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
     Similarly, only stations that record flood depth in centimeters are recorded into the dataframe. Additionally, only columns *station_id*, *Longitude*, *Latitude*,	and *SIUnit* are read into the dataframe. 18 entries with abnormal longitude and latitude values are removed after being identified via human inspection of the data. The following table is a snapshot of the first five entries of the flood sensor data, with a total of 1,965 entries after dropping duplicates:

      | station_id                               | Longitude  | Latitude   | SIUnit |
      |------------------------------------------|-----------|-----------|--------|
      | 648c0721-9ae3-4a3b-9007-31dd06a5f293     | 120.241250 | 23.450130 | cm     |
      | b320d298-d3aa-4954-874a-79696f550efa     | 120.188995 | 23.428696 | cm     |
      | c7c0c173-be6c-4fd2-b743-921d987e7330     | 120.392550 | 23.483112 | cm     |
      | bc5af470-def9-4712-95da-8cc29c35fd60     | 120.160995 | 23.508854 | cm     |
      | 54c2b021-edc6-418f-bff5-ec96067b24e6     | 120.433920 | 23.441912 | cm     |

     *Sample snapshot of flood sensor dataset*

  - #### Geographical Data (SHP file): Mixed, one row per one unique town or row per one unique district
    To aggregate longitude and latitude data into district boundary data, a SHP file recording the geographical boundaries of each district in Taiwan is used. In the context of Taiwan's administrative districts, there are three levels: county-level, town-level, and village-level. In this project, the term "district" is used with the semantics of "county name + town name." The following table is a snapshot of the first three entries of the SHP file, with a total of 7,953 entries with no duplicates:

    | VILLCODE       | COUNTYNAME | TOWNNAME | VILLNAME  | VILLENG         | COUNTYID | COUNTYCODE | TOWNID   | TOWNCODE  | NOTE   | Name                | geometry                                       |
    |---------------|-----------|---------|-----------|----------------|----------|------------|---------|----------|-------|-------------------|------------------------------------------------|
    | 10013030S01  | 屏東縣    | 東港鎮  | None      | None           | T        | 10013      | T03     | 10013030 | 未編定村里 | 屏東縣東港鎮       | POLYGON ((120.48059 22.42686, 120.4764 22.4289...) |
    | 64000130006  | 高雄市    | 林園區  | 中門里    | Zhongmen Vil.  | E        | 64000      | E13     | 64000130 | None  | 高雄市林園區中門里 | POLYGON ((120.36757 22.51419, 120.36775 22.514...) |
    | 64000130008  | 高雄市    | 林園區  | 港埔里    | Gangpu Vil.    | E        | 64000      | E13     | 64000130 | None  | 高雄市林園區港埔里 | POLYGON ((120.38662 22.50019, 120.38652 22.500...) |

    *Sample snapshot of SHP file*

  - #### District Area Data: One grain per one unique district
    To aggregate and calculate flood damage on a district level, area information of each district is required to calculate flood damage. The following table is a snapshot of the first five entries of the area file, with a total of 368 entries.

    | district     | area      |
    |-------------|----------:|
    | 新北市板橋區 | 23137300 |
    | 新北市三重區 | 16317000 |
    | 新北市中和區 | 20144000 |
    | 新北市永和區 |  5713800 |
    | 新北市新莊區 | 19738300 |

    *Sample snapshot of the area dataset*

  - #### Data Schema
    As a result, after joining and removing unused columns, we obtain the following data schema:
    ![Schema](/assets/model.png)
    *Graphical Representation of Data Schema*
    
- ### Flood Event Identification
  - #### Thresholds and Definitions
    In this project, an individual flood event is defined by the following three criteria:
    1. Has an average flood depth exceeding 50 centimeters
    2. Has a time gap of at least 24 hours between its consecutive closest flood event on a district level
    3. Not exceeding a max flood depth of three meters 

    Flood events that do not meet the first and second criteria will not be identified as flood events; flood events occurring within 24 hours in the same district would be categorized as the same flood event. The reason that floods are grouped on a district level is that on a village level, flood events occuring in neighboring, small villages would be identified as the same flood event, which may not be accurate in terms of identifying individual flood events in practice. 

  - #### Processing Pseudocode
    The following pseudocode is used to sort, identify, and label individual flood events from the aforementioned data schema, creating a new dataframe flood_incidents:

    ```
    # Define thresholds parameters
    TIME_GAP_THRESHOLD = 24 hours
    DEPTH_THRESHOLD = 50
    DEPTH_OUTLIER = 300
    
    # Identify flood incidents within each district
    FOR each district in sorted dataframe by ['district', 'timestamp']:
    Compute time difference 'time_diff' between consecutive timestamps
    Mark a new incident if 'time_diff' > TIME_GAP_THRESHOLD and create
    a new Boolean variable 'new_incident'
    
    # Assign an incident group number within each district
    FOR each district:
    Compute cumulative sum of 'new_incident' to create
    'incident_group'
    
    # Generate unique incident_id for each incident
    FOR each (district, incident_group):
    Create a unique incident key using [earliest timestamp + district]
    Convert incident key to a numeric 'incident_id.'
 
    # Aggregate flood data at the incident level
    FOR each incident:
    Compute:
    - start_time (earliest timestamp)
    - end_time (latest timestamp)
    - min_flood_depth (minimum depth value)
    - max_flood_depth (maximum depth value)
    - avg_flood_depth (average depth value)
    - area, county name, town name, geometry, village number factor (of each district), village name
    
    # Filter out incidents that are outliers or below depth threshold
    REMOVE incidents where:
    max_flood_depth >= DEPTH_OUTLIER
    OR avg_flood_depth <= DEPTH_THRESHOLD
    
    # Return final processed flood incidents dataset
    ```

    The the following is a snapshot of the first three entries final resulting dataframe:

    | district     | incident_id | start_time          | end_time            | min_flood_depth | max_flood_depth | avg_flood_depth | area      | county  | town  | vil  | geometry | factor |
    |-------------|------------|---------------------|---------------------|-----------------|-----------------|-----------------|-----------|--------|------|------|----------|--------|
    | 嘉義市東區  | 268        | 2022-10-01 09:00:00 | 2022-10-01 10:27:48 | 294.2           | 295.1           | 294.900000      | 30155600  | 嘉義市  | 東區 | 仁義里 | POLYGON ((120.45899 23.4542, 120.45889 23.4541...) | 39     |
    | 嘉義縣六腳鄉 | 462        | 2020-03-21 20:09:29 | 2020-03-21 20:09:29 | 77.3            | 77.3            | 77.300000       | 62261900  | 嘉義縣  | 六腳鄉 | 古林村 | POLYGON ((120.28176 23.49113, 120.28066 23.491...) | 25     |
    | 嘉義縣六腳鄉 | 464        | 2020-04-13 21:26:57 | 2020-04-13 21:26:57 | 75.7            | 75.7            | 75.700000       | 62261900  | 嘉義縣  | 六腳鄉 | 古林村 | POLYGON ((120.28176 23.49113, 120.28066 23.491...) | 25     |

    *Sample snapshot of the dataframe flood_incidents*
    
    Note that the feature *factor* is also created in this step, representing the number of villages in the corresponding district. This factor will be used to adjust the flood damage value, which will be introduced in the next section.

- ### Flood Damage Estimation
  - #### Methodology
    The methodology applied in this project to estimate flood damage is derived from the Global flood depth-damage functions: Methodology and the database with guidelines by the European Commission[1]. The high-level idea of this methodology is to scale down the MaxDamage value by a fraction, which is determined by a scaling parameter. In practice, this research proposed a generalized approach to estimating flood damage based on the following mathematical relation:
    
    $$UnitEstimatedDamage = D(m) \times MaxDamage$$ 
    
    Specifically, $D\left(m\right)$ is a function that takes flood depth in meters as an input and outputs a real value that is between 0 and 1. In practice, only nine points on the function are provided in the research, as opposed to the function itself. Therefore, when dealing with values that do not land on the provided points, the interpolate function in numpy is used to derive the function value given the flood depth:

    $$D(m) \in [0,1]$$

    MaxDamage is a predetermined parameter in Euros/square meters that changes case by case along with the country and the land usage. In this project, due to not having access to building-level data, the parameter of High Income Contries/Commercial/Land-use Based with a value of 309 is used.

  - #### Estimating Accumulated Damage
    To estimate the accumulated flood damage in a particular district $d$, we need to calculate the $area_d$ of the district $d$. However, due to the large area of districts, it is unrealistic to assume that each flood event has an impact on the entire district. Therefore, the approach used in this project is to view flood events as village-level events, then divide the entire district area by the number of villages in the district. Specifically, the four following hypotheses are made in order to estimate the flood damage in each district:
    1. Each flood event reflects a flood event on a village-level
    2. Each flood event is a uniform distribution of flood depth within the corresponding village, and all villages are homogeneous entities
    3. Village area can be proxied by dividing the area of the district by the number of villages in the district
    4. All flood damage is commercial damage\
       
    Following the aforementioned hypotheses, given a flood event $i$ with a uniform flood depth $m_i$, a village $v$ in district $d$ with area $area_v$ has a flood damage of:

    $$Damage_{v, i} = D(m_i) \times 309 \times area_v$$

    In which $area_v$ is:

    $$area_v = \frac{area_d}{n_d}$$

    in which $n_d$ is the number of villages in district $d$. Therefore, to accumulate the flood damage in district $d$, given the set of villages $V$ and the set of flood events $I$ in district $d$, the total accumulated flood damage in district $d$ is:

    $$Damage_d = \sum_{v \in V} \sum_{i \in I} D(m_i) \times 309 \times \mathrm{area}_v$$

    To convert the value of Euros in 2010 to New Taiwan Dollars in 2025, the following conversion based on CPI and exchange rate is applied to the final accumulated damage:

    $$Damage_{NTD,2025} = Damage_{€,2010} \times \frac{EUCPI_{2025}}{EUCPI_{2010}} \times E_{€,NTD}$$

    As a result, we can obtain the adjusted damage of each flood event in the form of the following dataframe:
        | District      | Incident ID | Start Time           | End Time             | Min Flood Depth | Max Flood Depth | Avg Flood Depth | Area        | County  | Town   | Village | Geometry | Factor | Estimated Damage | Estimated Damage Adjusted |
    |--------------|------------|----------------------|----------------------|----------------|----------------|----------------|-------------|---------|--------|---------|----------|--------|------------------|--------------------------|
    | 嘉義市東區    | 275        | 2022-10-01 09:00:00  | 2022-10-01 10:27:48  | 294.2          | 295.1          | 294.9          | 30155600.0  | 嘉義市   | 東區    | 仁義里   | POLYGON ((120.45899207200011 23.45419898600005...) | 39.0    | 2.087919e+08     | 9.826027e+09               |
    | 嘉義縣六腳鄉  | 469        | 2020-03-21 20:09:29  | 2020-03-21 20:09:29  | 77.3           | 77.3           | 77.3           | 62261900.0  | 嘉義縣   | 六腳鄉  | 古林村   | POLYGON ((120.28175757500003 23.49113104400004...) | 25.0    | 3.596602e+08     | 1.692609e+10               |
    | 嘉義縣六腳鄉  | 471        | 2020-04-13 21:26:57  | 2020-04-13 21:26:57  | 75.7           | 75.7           | 75.7           | 62261900.0  | 嘉義縣   | 六腳鄉  | 古林村   | POLYGON ((120.28175757500003 23.49113104400004...) | 25.0    | 3.557201e+08     | 1.674066e+10               |

    *Sample snapshot of the final dataframe*

- ### Visualization
  The final dataframe is exported and imported to Tableau for building the dashboard  

  <div class='tableauPlaceholder' id='viz1739738786921' style='position: relative'><noscript><a href='#'><img alt='Dashboard ' src='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Ta&#47;TaiwanFloodMapVisalization&#47;Dashboard&#47;1_rss.png' style='border: none' /></a></noscript><object class='tableauViz'  style='display:none;'><param name='host_url' value='https%3A%2F%2Fpublic.tableau.com%2F' /> <param name='embed_code_version' value='3' /> <param name='site_root' value='' /><param name='name' value='TaiwanFloodMapVisalization&#47;Dashboard' /><param name='tabs' value='no' /><param name='toolbar' value='yes' /><param name='static_image' value='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Ta&#47;TaiwanFloodMapVisalization&#47;Dashboard&#47;1.png' /> <param name='animate_transition' value='yes' /><param name='display_static_image' value='yes' /><param name='display_spinner' value='yes' /><param name='display_overlay' value='yes' /><param name='display_count' value='yes' /><param name='language' value='en-US' /></object></div>                <script type='text/javascript'>                    var divElement = document.getElementById('viz1739738786921');                    var vizElement = divElement.getElementsByTagName('object')[0];                    if ( divElement.offsetWidth > 800 ) { vizElement.style.width='1000px';vizElement.style.height='827px';} else if ( divElement.offsetWidth > 500 ) { vizElement.style.width='1000px';vizElement.style.height='827px';} else { vizElement.style.width='100%';vizElement.style.height='927px';}                     var scriptElement = document.createElement('script');                    scriptElement.src = 'https://public.tableau.com/javascripts/api/viz_v1.js';                    vizElement.parentNode.insertBefore(scriptElement, vizElement);                </script>

  
- ### Limitations
  - **Sensor Coverage Bias:** Some regions lack adequate sensor installations, leading to potential underestimations.
  - **Simplified Damage Estimation:** The model assumes uniform damage intensity per region, which may not reflect real-world variations.
  - **Exclusion of Other Risk Factors:** Factors such as land use and population density are not fully incorporated.

- ### References
  [1] Water Resources Agency, Taiwan. "Flood Sensor Data." Accessed 2025.  
  [2] European Commission Joint Research Centre. "Global Flood Depth-Damage Functions." 2017.  
  [3] National Land Surveying and Mapping Center, Taiwan. "Administrative Boundary Shapefiles." Accessed 2025.  

https://vocus.cc/article/66a3839dfd89780001ccfee3

Hypothesis:
1. Only include flood sensors 與縣市政府合建
2. Only include flood sensors that use CM (detects depth)
3. An incident is Define an incident cutoff (e.g., if two consecutive records in the same county/district have a time gap >1 hour, they belong to different incidents).
