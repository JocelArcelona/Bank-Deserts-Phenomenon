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
  ■ The FDIC dataset contains information on all fdic-insured banks inside and outside the US (US States + US territories). 
Created a function to access the fdic api that can handle pagination due to the api’s limit per request. The function also handles saving all the data retrieved locally. 
  ■ The FDIC dataset needed to be geocoded to extract the census tract GEOID and locations. The geocoding process was done using the US 
Census Bureau’s Geocoding Services Web API. 
Created a function to batch geocode all bank addresses to retrieve geoid and coordinates 
○ Steps for data preprocessing: 
  ■ After geocoding the bank addresses, bank locations outside of the US were dropped 
  ■ Concatenate all csv files containing geocoded addresses of all banks inside the US and this was followed by the same basic practices during 
data cleaning like checking null values, dropping nulls, dropping 
duplicated rows and assuring that the data is in the correct data type. 
  ■ Merged the geocoded addresses columns with the original fdic bank 
dataset to obtain a dataset containing bank information with GEOID 
(feature engineered using State + County + Census tract code obtained 
from geocoding) and coordinates. 

**TIGER/line Shapefiles**
○ Zipped files containing geospatial data that provide detailed geographical boundary information (in the case of this analysis, census tract level geographical boundary information) 

○ Steps for data collection: 
  ■ Used beautifulSoup to parse through the html document and used 
webdrivermanager to download all zipped TIGER/line shapefiles from the US Census Bureau. This contains all census tracts geographical 
boundaries by state. 
○ Steps for data preprocessing: 
  ■ Concatenated all census tract shapefiles (56 shapefiles) into one 
GeoDataFrame and saved it as a parquet file to preserve data types and compress the geometry feature
  ■ Dropped US territories (outside of the US) based on their state code (60 for American Samoa, 66 for Guam, 69 for Northern Mariana Islands, 72 for Puerto Rico and 78 for US Virgin Islands) 
  ■ Merged the shapefiles data with the RUCA (Urban/Rural Classification) data on GEOID – this is crucial for identifying the bank desert status. 
Shapefiles data contain coordinates of census tracts, the RUCA contains the Community Type of each census tract. 

**USDA (Rural/Urban Classification using RUCA code) + Census Tract Relationship Files from the US Census**
○ The rural-urban commuting area code (RUCA code) classifies US census tracts on their level of urbanization, population density and daily commuting (this is for the 2010 version of RUCA). For 2020, the main criteria will be switched to Housing Density. 
○ This dataset is an outdated version (2010) – updated to 2020 using the census tract relationship files to match the GEOID for all the other datasets 
○ 2020 version from the USDA will be released on Spring 2025 

○ Steps for data collection: 
  ■ This dataset was provided by the USDA, it can be found on their website and can be downloaded as a csv file 
○ Steps for preprocessing: 
  ■ State, county and census tract codes could contain some 0’s in the beginning like 01 for Alabama. Due to the format of the file, it keeps 
dropping all the 0’s in front of the codes. To prevent this, I had to change the data type to string and use .zfill to maintain the nature of the codes (a full geoid on the census tract level contains 11 digits). 
  ■ Repeated the same process from above to the census tract relationship files data and merged both datasets on GEOID. This will update the 2010 GEOID from RUCA to the 2020 GEOID version – this will make merging the datasets easier and more aligned.
  
**FDIC DATASET + NCUA DATASET**
○ This dataset contains ALL bank data and credit unions (all geocoded, containing GEOID and coordinates) 

○ Steps for data collection: 
  ■ Developed a function to geocode bank and credit union locations using the FDIC API and NCUA data, then integrated the geocoded addresses back into the original FDIC and NCUA datasets to create a comprehensive bank dataset.
○ Steps for data preprocessing: 
  ■ Same preprocessing steps taken for the FDIC data. Rename columns, drop null values, drop duplicated rows and ensure correct data types. 

**US Census Data (ACS5) for Modeling** 
○ Steps for data preprocessing:
  ■ Prepared the census data for modeling by identifying and extracting the majority and minority race/age/gender categories based on their 
percentages (instead of each race, each age and gender categories as its own column, I summarized it to majority and minority race/age/gender 
and included the majority and minority percentages as well) 
  ■ Dropped columns that showed high correlation with each other to prevent multicollinearity and lessen feature space 

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
Key Features: Socioeconomic factors such as Poverty, Income, Population Density, Housing Units, etc.
Target Variable: Bank Desert Status
Bank Desert Status: Positive class: Bank Desert, Negative class: Not a Bank Desert (Potential Bank Desert + Not a Bank Desert)
Models Used: Logistic Regression, Decision Trees, Random Forest
Methodology: Train-test split, Scaling, Stratification, Cross-validation, Hyperparameter tuning with GridSearchCV (Optimization)
Evaluation: Metric - Weighted Recall
Results Interpretation: Feature Importance, SHAP Analysis

# Feature Importance and SHAP analysis visualization
![Screenshot (18)](https://github.com/user-attachments/assets/af45bd86-b64d-4f41-910d-b41a04ccdfae)
![summary_plot](https://github.com/user-attachments/assets/2ef1dbad-ac0d-4730-b14d-0d5dca816fbf)





