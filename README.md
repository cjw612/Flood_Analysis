# Flood Damage Analysis from 2019 to 2022 in Taiwan
Analysis of flood data from 2019 - 2022 in Taiwan

https://data.nat.gov.tw/dataset/7441 (sensor dataset)

https://maps.nlsc.gov.tw/pro/download.jsp (map shp file)

https://tradingeconomics.com/taiwan/consumer-price-index-cpi (taiwan cpi)

https://gis.rchss.sinica.edu.tw/qgis/archives/2823/ (Transverse Mercator EPSG)

https://cybsbox.cy.gov.tw/CYBSBoxSSL/edoc/download/68467 (10 cm cutoff)

Hypothesis:
1. Only include flood sensors 與縣市政府合建
2. Only include flood sensors that use CM (detects depth)
3. An incident is Define an incident cutoff (e.g., if two consecutive records in the same county/district have a time gap >1 hour, they belong to different incidents).
