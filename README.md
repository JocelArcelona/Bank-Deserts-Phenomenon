# Bank-Deserts-Phenomenon
![banking-desert-hero_900x506x4800-2700-0-0](https://github.com/user-attachments/assets/224c0f3d-c92b-4dc7-b124-c0862aecacf4)

# Problem Statement 
Access to physical banking services is often limited or non-existent in some census tract areas in the US, a phenomenon known as Bank Deserts. This study aims to analyze and understand Bank Deserts and the relationship between socioeconomic factors and the availability of banking services by integrating data from the US Census Bureau (Census data, TIGER/line shapefiles), FDIC (federally insured banks), NCUA (credit unions) and USDA. 

# What is a Bank Desert? 
A banking desert is a census tract without a bank branch located within a certain radius from its population center. The radius is determined by: 2 miles for Urban communities, 5 miles for Suburban communities and 10 miles for Rural communities 

# Objectives 
1) Feature engineer a target variable called Bank Desert Status
2) Create a classification model that predicts Bank Desert Status based on socioeconomic factors (income, population density, house units, etc.)
3) Create an interactive visualization showcasing census tracts filtered by state, county and census data + geographic distribution of branches

# Data Collection and Preprocessing (FDIC, TIGER/line shapefiles from the US Census, RUCA codes from the USDA, Census Tract Relationship Files from the US Census)
**FDIC DATASET: BankFind Suite API** 

○ This dataset contains all banks taken from the FDIC’s API 

○ Steps for data collection: 

  1) The FDIC dataset contains information on all fdic-insured banks inside and outside the US (US States + US territories). 
Created a function to access the fdic api that can handle pagination due to the api’s limit per request. The function also handles saving all the data retrieved locally. 
  2) The FDIC dataset needed to be geocoded to extract the census tract GEOID and locations. The geocoding process was done using the US 
Census Bureau’s Geocoding Services Web API. 
Created a function to batch geocode all bank addresses to retrieve geoid and coordinates 

○ Steps for data preprocessing: 

  1) After geocoding the bank addresses, bank locations outside of the US were dropped 
  2) Concatenate all csv files containing geocoded addresses of all banks inside the US and this was followed by the same basic practices during 
data cleaning like checking null values, dropping nulls, dropping 
duplicated rows and assuring that the data is in the correct data type. 
  3) Merged the geocoded addresses columns with the original fdic bank 
dataset to obtain a dataset containing bank information with GEOID 
(feature engineered using State + County + Census tract code obtained 
from geocoding) and coordinates. 

**NCUA DATASET: List of Active Federally Insured Credit Unions Hyperlink** 

○ This dataset contains all currently active credit unions that are federally insured

○ Steps for data collection: 

  1) The NCUA provides a [hyperlink](https://ncua.gov/files/publications/analysis/federally-insured-credit-union-list-december-2024.zip) that automatically downloads the Excel file when selected.
  2) Remove carriage returns from headers and save the file as CSV. 
  3) The city names were all truncated after 15 characters, so some manual cleaning was implemented to fix those cities.
  4) The NCUA dataset needed to be geocoded to extract the census tract GEOID and locations. The geocoding process was done using the US Census Bureau’s Geocoding Services Web API. Created a function to batch geocode all bank addresses to retrieve geoid and coordinates.

○ Steps for data preprocessing: 

  1) Merged the geocoded addresses columns with the original NCUA credit union 
dataset to obtain a dataset containing credit union information with GEOID 
(feature engineered using State + County + Census tract code obtained 
from geocoding) and coordinates. 

**TIGER/line Shapefiles**

○ Zipped files containing geospatial data that provide detailed geographical boundary information (in the case of this analysis, census tract level geographical boundary information) 

○ Steps for data collection: 

  1) Used beautifulSoup to parse through the html document and used 
webdrivermanager to download all zipped TIGER/line shapefiles from the US Census Bureau. This contains all census tracts geographical 
boundaries by state.

○ Steps for data preprocessing: 

  1) Concatenated all census tract shapefiles (56 shapefiles) into one 
GeoDataFrame and saved it as a parquet file to preserve data types and compress the geometry feature
  2) Dropped US territories (outside of the US) based on their state code (60 for American Samoa, 66 for Guam, 69 for Northern Mariana Islands, 72 for Puerto Rico and 78 for US Virgin Islands) 
  3) Merged the shapefiles data with the RUCA (Urban/Rural Classification) data on GEOID – this is crucial for identifying the bank desert status. 
Shapefiles data contain coordinates of census tracts, the RUCA contains the Community Type of each census tract. 

**USDA (Rural/Urban Classification using RUCA code) + Census Tract Relationship Files from the US Census**

○ The rural-urban commuting area code (RUCA code) classifies US census tracts on their level of urbanization, population density and daily commuting (this is for the 2010 version of RUCA). For 2020, the main criteria will be switched to Housing Density. 
○ This dataset is an outdated version (2010) – updated to 2020 using the census tract relationship files to match the GEOID for all the other datasets 
○ 2020 version from the USDA will be released on Spring 2025 

○ Steps for data collection: 

  1) This dataset was provided by the USDA, it can be found on their website and can be downloaded as a csv file
     
○ Steps for preprocessing: 

  1) State, county and census tract codes could contain some 0’s in the beginning like 01 for Alabama. Due to the format of the file, it keeps 
dropping all the 0’s in front of the codes. To prevent this, I had to change the data type to string and use .zfill to maintain the nature of the codes (a full geoid on the census tract level contains 11 digits). 
  2) Repeated the same process from above to the census tract relationship files data and merged both datasets on GEOID. This will update the 2010 GEOID from RUCA to the 2020 GEOID version – this will make merging the datasets easier and more aligned.
  
**FDIC DATASET + NCUA DATASET**

○ This dataset contains ALL bank data and credit unions (all geocoded, containing GEOID and coordinates) 

○ Steps for data collection: 

  1) Developed a function to geocode bank and credit union locations using the FDIC API and NCUA data, then integrated the geocoded addresses back into the original FDIC and NCUA datasets to create a comprehensive bank dataset.
     
○ Steps for data preprocessing: 

  1) Same preprocessing steps taken for the FDIC data. Rename columns, drop null values, drop duplicated rows and ensure correct data types. 

**US Census Data (ACS5) for Modeling** 

○ Steps for data preprocessing:

  1) Prepared the census data for modeling by identifying and extracting the majority and minority race/age/gender categories based on their 
percentages (instead of each race, each age and gender categories as its own column, I summarized it to majority and minority race/age/gender 
and included the majority and minority percentages as well) 
  2) Dropped columns that showed high correlation with each other to prevent multicollinearity and lessen feature space 

# Feature Engineering 
1) Bank Desert Status (Target Variable): contains the bank desert status of each census tract in the US (bank desert, potential bank desert, not a bank desert)
2) Land Area in Sq Miles: Converted ALAND column (area in meters) to area in square miles 
3) Population Density: Population (from US Census data) divided by Land Area in Sq Miles 
4) Housing Density: House Units (from US Census data) divided by Land Area in Sq Miles 
5) Coordinates, State code, County code, Census Tract code: Obtained additional features by geocoding bank addresses 
6) Longitude, Latitude: Obtained from splitting the coordinates feature 
7) Community Type (Urban, Rural and Suburban): Given RUCA codes by census tract, I applied a lambda function on the Primary RUCA code column based on the ruca code description provided. 
If the ruca code is [1, 2, 3] – Urban. [4, 5, 6] – Suburban, and Rural for everything else. 
8) Radius Threshold in Miles: A criteria defined by the federal reserve board. 2 miles for urban communities, 5 miles for suburban communities and 10 miles for rural communities. I just mapped the Community Type using these predefined radius thresholds to create this feature. 
9) Bank Name: Created this feature by taking a main branch’s name + branch name if the main office is not equal to branch name. If the main branch is equal to the branch name, maintain the same name. 
10) GEOID for banks: During geocoding of the bank addresses, the state code, county code and census tract code were separated. Obtained GEOID by applying a lambda row function to concatenate all three columns as one. All the other datasets contained GEOID already.

# Bank Desert Status 
Created a function that determines the bank desert status of census tracts using a shapefile and bank location data.

**Extract Key Data:**
Iterates through census shapefile rows to extract GEOID (11-digit FIPS code), Community Type, Radius Threshold (2 miles for Urban, 5 for Suburban, 10 for Rural), and INTPTLAT/LON (internal point coordinates).
Community Type is based on USDA classifications, and the radius threshold follows FDIC guidelines.

**Filter Bank Data:**
Filters the bank dataset by GEOID, as a tract may contain multiple banks.

**Calculate Distances:**
Computes the geodesic distance (miles) between each census tract's internal point and nearby bank branches.

**Classify Bank Desert Status:**

0 banks within radius → Bank Desert

1 bank within radius → Potential Bank Desert

2+ banks within radius → Not a Bank Desert

**Update DataFrame:**
Assigns the classification to the census shapefile dataframe and returns the updated dataset.

# Bank Desert Classification using Socioeconomic Factors (for Identifying Financially Underserved Communities -- Bank Deserts) 
**Key Features:** Socioeconomic factors such as Poverty, Income, Population Density, Housing Units, etc.

**Target Variable:** Bank Desert Status

**Bank Desert Status:** Positive class: Bank Desert, Negative class: Not a Bank Desert (Potential Bank Desert + Not a Bank Desert)

**Models Used:** Logistic Regression, Decision Trees, Random Forest

**Methodology:** Train-test split, Scaling, Stratification, Cross-validation, Hyperparameter tuning with GridSearchCV (Optimization)

**Evaluation:** Metric - Weighted Recall

**Results Interpretation:** Feature Importance, SHAP Analysis

# Feature Importance and SHAP analysis visualization
**Top 3 most influential Features to Bank Desert prediction according to Feature Importances:** 
1) Population Density
2) House Units
3) OwnOcpHous%
![Screenshot (18)](https://github.com/user-attachments/assets/af45bd86-b64d-4f41-910d-b41a04ccdfae)

Extracting feature importances from our random forest classifier does not necessarily equate to extracting the best predictors in the model. It just tells us which features were the biggest influence to the target variable but we dont know whether this influence positively contributed to predicting bank deserts or the other way around. This is where SHAP analysis comes into play.
The beeswarm plot of the positive class below shows not just the importance of each feature but their actual relationship with bank desert prediction for the entire population. 
![summary_plot](https://github.com/user-attachments/assets/2ef1dbad-ac0d-4730-b14d-0d5dca816fbf)

**To summarize the top 3 features based on the beeswarm plot above:**
1) Census Tracts with lower population density values lead to more bank deserts
2) Census Tracts with lower housing unit values lead to more bank deserts
3) And inversely, Census Tracts with higher homeownership percentage lead to more bank deserts.

# Top 2 most influential features (according to feature importance and shap analysis) line plot 
![Screenshot (24)](https://github.com/user-attachments/assets/d3d24425-37ca-4089-a720-84bfc99a6287)
**Key Takeaways:**
- Urban areas with Potential Bank Desert status have the highest density, which is unusual compared to suburban and rural trends. The trend of bank desert status is not as clear for Urban areas potentially due to urban areas having a high population density by definition so it's harder to see the pattern compared to Rural and Suburban communities. 
- For suburban and rural areas, the trend is clear: Bank Deserts are in the least dense locations, while Not a Bank Desert areas have the highest densities.
- This suggests that in suburban and rural areas, banking presence is linked to higher population density, but in urban areas, other factors might influence bank desert status.

![Screenshot (25)](https://github.com/user-attachments/assets/5fd46d3c-488f-4d50-888a-e06ca90309a7)
**Key takeaways:**
- Across all community types, areas with more housing units are less likely to be Bank Deserts.
- The difference in median housing units between "Bank Desert" and "Not a Bank Desert" areas is most pronounced in rural areas.
- This suggests a strong relationship between banking access and housing concentration, where fewer housing units might correlate with lower demand for traditional banking services.
- Compared to population density, housing units provide a clearer and more consistent pattern in defining bank desert status across all community types. The inconsistencies seen in population density—especially in urban areas—suggest that housing units might be a better predictor of banking access.
- The USDA, which manages the Rural-Urban Classification (RUCA), also shifted to using housing density instead of population density in the 2020 version of the framework. This reinforces the idea that housing units may be a more stable measure for defining community types and, by extension, banking access.

# Interactive Census Tract Map
This interactive map, built with Leaflet and Shiny, allows users to explore census tracts by state and county. It visualizes census data (like poverty and income) using a continuous color scale, similar to a density heatmap. Users can filter by region and hover over tracts to view details like geoid, tract name, bank desert status, and other categorical information. The map helps identify trends in socioeconomic conditions and banking access.

**Note:**
Run census_tracts_viz_r rmd file found under the visualization directory to see the full interactive visualization 
![Screenshot (22)](https://github.com/user-attachments/assets/86786e50-7ad4-45e4-9633-09ca3c32b078)


# Interactive Bank Desert and Locations Maps
These maps, built with Folium, allow users to visualize the geographic distribution of bank deserts in each state by census tract, as well as individual bank locations. Users can hover over specific census tracts to view Census demographic and socioeconomic characteristics, or hover over branch branch locations to view the bank name.

**Note:**
Maps for all 50 states are created using the Folium_Map_by_State.ipynb. Users can alternatively create a map for a single state using Foluim_Map.ipynb, which is preset to California.
![Bank Desert Map](https://github.com/user-attachments/assets/b45027e5-6919-447f-9f88-c6594d3b0658)














