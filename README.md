# Discovering Knowledge for the City of Charlotte, NC
This project is a course requirement for DSBA-6162, Knowledge Discovery in Databases, at UNCC for the Fall semester, 2021.

**Team Members:**

###### Naomi Thammadi
###### Kevin Gharavizadeh
###### Imad Ahmad
###### Dustin Ballentine


## Project Motivation
The city of Charlotte has, like many cities, made large volumes of data open to the public in an online repository. This open data provides substantial potential for the city to benefit from knowledge discovery and insights generated by members of the public, such as ourselves. Service requests dialed to 311 represent a large opportunity to delve into the needs of the population and potentially extract useful trends that could allow the city to improve its service to its citizens. The goal of this project is to explore and uncover exactly those trends in the hope that the knowledge we discover may be used by the city to improve the quality of life of our families, friends, and neighbors.
## Research Question
Do different areas within the city of Charlotte experience higher recurrence of any type of 311 service request than other areas, and do those areas correlate with red-line districts or other known demographic or socioeconomic profiles?
## Data Resources
<ol>
<li> 311 Service Requests. </li> Data was retrieved from the City of Charlotte's
<a href="https://data.charlottenc.gov/datasets/charlotte::service-requests-311/about"> Open Data Portal </a>
on 30 September, 2021. The data set is a collection of the public records requests received by the City of Charlotte from 2017 through March, 2020. It contains multiple descriptive fields related to service request entry, including request type, location (street, lat/long, etc.), date, city department responsible for the request, and point of entry.
<li> Census Household Income Data </li> Census data was taken from <a href="https://data.census.gov/cedsci/table?d=ACS%205-Year%20Estimates%20Data%20Profiles&hidePreview=false&vintage=2018&layer=zcta5&tid=ACSDP5Y2019.DP03">the datasets of census.gov</a>. This data set is a collection of household income statistics, Employment statistics, Details about Families and Non-families, People under poverty line and People commuting to work. The dataset consists of 932 rows and 551 features.
<li> Census Tract Ids and Zip Codes </li> The source of Census Tract Id to Zip code conversion has been taken from <a href="https://www.huduser.gov/portal/datasets/usps_crosswalk.html#data">the dataset portal of huduser.gov</a>. This conversion was necessary to link the census data to 311-Service Requests data. The dataset has 5301 rows and 6 features.
<li> CMPD Incidents Crime Data </li> Data was retrieved from the City of Charlotte's <a href="https://data.charlottenc.gov/datasets/charlotte::cmpd-incidents/about"> Open Data Portal </a> on 30 September, 2021. The dataset is a collection of CMPD Incidents reported from 2017-2021. The dataset has both violent and non-violent crimes reported. Any incident reported with NIBRS Code 800 and above is considered non-violent. The dataset is grouped using the Neighborhood Profile Area, month and year to create a CRIME_SCORE.

</ol>

## Python libraries to install
Before running any of the Jupyter notebooks mentioned in the repository, the below libraries need to be installed in the local machine using Anaconda:

```
    pip install missingno
    pip install geopandas
    pip install contextily
```

## Data Preprocessing
### 311 Service Requests
From the initial 311 service requests dataset, we dropped 15 features which were either redundant or contained too many missing values. We also dropped just over 99,000 records that contained null values for address, zipcode, and LAT/LON coordinates, since those are the fields of primary interest to our analysis, we have no reasonable way to impute those fields, and that represented only about 5.5% of the data. Finally, we dropped an additional 8,279 records which pertained to COVID-19 protocols, Solid Waste Services administrative functions such as a data upload, and 8 additional call types which had fewer than 10 total occurrences. <br>
Moving on with the 14 remaining features, we transformed two features and engineered another five as discussed below:
##### Transformations
<ol>
  <li> RECEIVED_DATE - Simple datatype transformation from object to datetime </li>
  <li> FULL_ADDRESS - Replaced with a rank-ordered index, with 1 representing the address with the highest call volume, ascending in order inversely to the volume of 311 calls. </li>
</ol>

##### Engineered Features
<ol>
<li> REQUEST_CAT - The original data contained 165 separate types of 311 service calls. Many of the categories contained only a handful of records, and many were clearly related to other call types. We therefore decided to bin the call types into a new feature called request categories. Twenty-three of the types were imported directly as categories--mainly the largest categories or those without a clear connection to other types. The remaining call types were binned into 16 categories, yielding 39 final categories. The breakdown of which types were binned into which categories can be seen in the <a href="https://github.com/DABallentine/knowledge_discovery_charlotte/blob/main/Jupiter%20Notebooks/EDA_and_Preprocessing.ipynb"> EDA and preprocessing notebook </a>.  </li>
  <li> RECEIVED_YEAR - Parsed out the year from the DATE_RECEIVED feature </li>
  <li> RECEIVED_MONTH - Parsed out the month from the DATE_RECEIVED feature </li>
  <li> SEASON - Parsed out the quarter from the DATE_RECEIVED feature, which generally differs from the official season of the year by only 1 week </li>
  <li> TOTAL_CALLS - Summed the total number of calls from each address </li>
  <li> COL_MERGE_INDEX - Combined NEIGHBORHOOD_PROFILE_AREA, RECEIVED_MONTH and RECEIVED_YEAR for merging 311-Service Requests with CMPD Crime dataset </li>
  <li> HISTORIC_REDLINING - Categorize Neighborhood Profile areas based on historic redlining of Charlotte city in 1935</li>
</ol>

### Census Income Data
The dataset consisted of 551 features that were reduced to 24 features as part of the data preprocessing which can be seen in the <a href="https://github.com/DABallentine/knowledge_discovery_charlotte/blob/main/Jupiter%20Notebooks/Census_Data_Preparation.ipynb"> Census Data Preparation Notebook</a>. 

##### Transformations
<ol>
  <li>GEO_ID - The original data from the GEO_ID column is in the format 1400000US37119000100. This data has been parsed to only include the 11-digit Tract Id after 'US'.</li>
</ol>

##### Engineered Features
<ol>
  <li>PERCENT EMPLOYED_In labor force - Percentage of Employed in labor force to the total employed population</li>
  <li>PERCENT EMPLOYED_Not in labor force - Percentage of Employed not in labor force to the total employed population</li>
  <li>PERCENT EMPLOYED_Female Only - Percentage of Females Employed to the total employed population</li>
  <li>PERCENT HOUSEHOLD INCOME_Lower Income Households - Feature created by adding up the Income and Benefits of households < $10,000 and between $10,000 to $34,999 then dividing by Total households</li>
  <li>PERCENT HOUSEHOLD INCOME_Mid Income Households - Feature created by adding up the Income and Benefits of households between $35,000 to $149,999 then dividing by Total households</li>
  <li>PERCENT HOUSEHOLD INCOME_Higher Income Households - Feature created by adding up the Income and Benefits of households between $150,000 to $199,999 and > $200,000 then dividing by Total households</li>
  <li>PERCENT HOUSEHOLD INCOME_Retired Householders - Feature created by dividing number of retired households by  Total households</li>
  <li>PERCENT COMMUTING TO WORK_By Car - Feature created by adding up the commuting to work alone in car and commuting to work by carpool then dividing by all ways of commuting to work</li>
  <li>PERCENT COMMUTING TO WORK_Public transportation - Feature created by dividing count commuting by Public Transportation by all ways of commuting to work</li>
  <li>PERCENT COMMUTING TO WORK_Walk - Feature created by dividing count commuting by Walk by all ways of commuting to work</li>
  <li>PERCENT COMMUTING TO WORK_Other - Feature created by dividing count commuting by Other means by all ways of commuting to work</li>
  <li>PERCENT COMMUTING TO WORK_Worked at home - Feature created by dividing count of people working from home by all ways of commuting to work</li>
  <li>PERCENT INSURED_Population with health insurance - Feature created by dividing count of people having health insurance by total count of population</li>
  <li>PERCENT INSURED_Population without health insurance - Feature created by dividing count of people not having health insurance by total count of population</li>
  <li>ZIP_YEAR - Combined ZIP_CODE and YEAR column to map the data accordingly to the Service Requests dataset</li>
</ol>

### CMPD Incidents Crime Data
The preprocessing of this dataset can be seen in the <a href="https://github.com/DABallentine/knowledge_discovery_charlotte/blob/main/Jupiter%20Notebooks/Census_Data_Preparation.ipynb"> Crime_Data_Preparation Notebook</a>.  

##### Engineered Features
<ol>
  <li> MONTH - Parsed out the month from the DATE_REPORTED feature </li>
  <li> COL_MERGE_INDEX - Combined NPA, MONTH and YEAR for merging with 311-Service Requests</li>
  <li> INCIDENT_COUNT - Total count of incidents reported per NPA every month </li>
  <li> CRIME_SCORE - a function of INCIDENT_COUNT divided by the total incidents reported in that month and then multiplied by 100 to calculate the percentage</li>
</ol>

## Data Understanding and Exploration
### Overview
##### Categories
Non-Recyclable Items make up over half of the requests and Recyclable Items comprise another 13%. Binning into the 39 categories created a much more even balance through categories 3-16, with each comprising from 7% down to 1% of total requests, as opposed to the inital proportions which were mostly much less than 1%. Note that the scale on the graph below is logarithmic to compensate for the imbalance between categories.
![image](https://user-images.githubusercontent.com/78170609/139606719-8fd9a565-d720-4926-acdf-b705106adb1b.png)

### Time Series and Seasonality
Plotting the requests over various time intervals seems to show a steady increase in call volume by year (2016 and 2021 are only partially represented in the data), with 2020 perhaps breaking the trend due to the COVID-19 pandemic. The summer months have the highest volume of requests, particularly in June and July, and there exists a clear trend for the most calls on Monday each week, and then tapering off as the week goes on. Daily, calls are fairly uncommon in the mornings, which was somewhat surprising, peaking in the early afternoon and then decreasing drastically after dinner time.
![image](https://user-images.githubusercontent.com/78170609/139607530-862f1398-e667-4266-9db6-6e5bc99e65fd.png)

We do observe some differences by category when plotting over different timeframes. For example, plotting by season across all categories reveals several categories with peaks during the winter months. Though not as extreme, each of the other seasons does also have one category somewhat above the even 25% split.
![image](https://user-images.githubusercontent.com/78170609/139609372-6f7c2050-60f9-4b0e-9f53-381fa8bd7090.png)
![image](https://user-images.githubusercontent.com/78170609/139610261-0550828e-91cf-4047-b74f-985c6873624b.png)

The category with the greatest disparity, SW ONLY - DOOR HANGER LEFT, seems to represent a note left on someone's door by the Solid Waste department. Possible reasons for this trend include a large proportion of the population traveling during the winter months. A closer look reveals that the peak actually occurs in February and March, so it does not seem to correlate to the Christmas or New Year holidays. Without additional data going deeper into the nature of this category, it is difficult to surmise what the trend may represent. FIELD OBSERVED PROBLEMs, almost 50% of which occur during Winter, also occur at a similarly high rate into April, totaling 73% of the year's records just in the first 4 months. This trend could provide a starting point for future research. What are the nature of those field-observed problems? Are there simply more city employees driving around during the early months of the year, or is there some seasonal effect pertaining to winter weather perhaps? We could perhaps answer those questions with the right data added to this analysis.
Several of the other categories align with common sense. For example, in Spring people return to the city parks, and issues which have cropped up unnoticed over the Winter months are identified and reported. Likewise, as the weather warms and the plants grow, people trim hedges, clean out and re-landscape old areas, and thus have more yard waste than at other times during the year.
![image](https://user-images.githubusercontent.com/78170609/139609399-f1cf45e5-3be6-4aa8-8609-2950fb97cd0b.png)

The monthly top categories by proportion reveal a few other imbalances across categories. 

### Customer Observations
By using each unique address as a "customer", we see one address that is 75% higher than the next highest address by call volume, and thereafter a fairly steady decrease across all addresses. The number one address corresponds to the Sharon Lakes Comdominiums complex located in Starmount Forest, in south Charlotte. The next few addresses also appear to be townhome developments or apartment complexes at various locations across the city. However, the #4 address is in close proximity to the #1 address, which may represent an area of high call volume worth investigating.
![image](https://user-images.githubusercontent.com/78170609/139608857-f3ceb422-2855-4372-ba61-d9d996cdb6a6.png)

### Geographic Analysis
Since our 311 Call data contains geospatial data, we decided to map a few data points onto the Charlotte map and see if we can spot some trends. A few variables we have explored are Recycle and Non Recycle requests (comparing 2016, 2018, 2020). 

For all our charts, please visit our Geopandas notebook here: <a href="https://github.com/DABallentine/knowledge_discovery_charlotte/blob/main/EDA_and_Preprocessing_with_MAPS.ipynb"> EDA and preprocessing with MAPS notebook </a>

The Charlotte Zip code shape can be found at the <a herf="https://data.charlottenc.gov/"> Charlotte Open Data</a> website.

Below are a few of our observations:

![image](https://user-images.githubusercontent.com/70532006/145228084-0a841d8e-16dd-4634-94f6-d9e4c5258060.png)

![image](https://user-images.githubusercontent.com/70532006/145228128-354637d5-0b3d-470b-ad8f-dab15e3f3681.png)

![image](https://user-images.githubusercontent.com/70532006/145228182-82dd2a38-7feb-4839-9f88-2164bab95d74.png)

Historic Red Lining Districts
![image](https://user-images.githubusercontent.com/70532006/145226286-028c9762-71a0-4be9-bf2c-febcf2f0350b.png)


## Data Preparation for Modeling

Before we can begin most of these techniques, we will need to transform our existing data into a form suitable for the algorithm to be applied.
Currently, our data exists in this form:

![image](https://user-images.githubusercontent.com/78170609/145226577-8fd024a3-8b5f-4fe4-8fff-9209e963769b.png)

Each record in the data represents a single 311 request. However, as stated in our research question, we are interested in whether “different areas within the city of Charlotte experience higher recurrence of any type of 311 service request than other areas” and then exploring any correlations between an area’s 311 service request profile and any “red-line districts or other known demographic or socioeconomic profiles”. Thus, the entity (i.e. record) of primary interest to us is the area, not the single request, and we will need to generally transform the data into the form below:

![image](https://user-images.githubusercontent.com/78170609/145226635-0e01ddd3-8322-475b-8fe6-6a0b8a7d9ee7.png)

### Association Rule Mining
Two new features are created to assign the predominant income levels and crime levels that the request neighborhood belongs to as PREDOMINANT_INCOME_BRACKET and CRIME_INDEX. Service Requests data is arranged as a matrix using a MERGE_INDEX (feature created as a combination of month, year and neighborhood profile area) and REQUEST_CAT, with the cell value of the matrix being the count of request raised based on the type, in a neighborhood, on a monthly basis. Since Association rules only accepts binary data (0 or 1), any cell having a value 0 is used as it is, and any cell having a value greater than 0 is used as 1. This format resembles the shopping basket format required to successfully generate association rules

### Predictive Modeling
Data preparation began very similar to that for association rules, with each record representing a monthly summary of a Neighborhood Profile Area rather than an individual 311 request. The key difference from association rule data is that the records are not binary, rather aggregate statistics of 311 calls. We compute 4 key aggregate statistics by month for each of the 39 service request categories: total volume, standard deviation, distribution, and proportion of late requests. Total volume for a NPA for a given month is divided by that neighborhood's population to yield a per capita volume, and then multiplied by 1000 to represent number of requests per thousand residents. Standard deviation is calculated for the month's distribution of daily request totals. The distribution variable is simply the number of unique addresses that supplied the 311 requests for each category for that month. Finally, the late variable is the proportion of requests that were submitted between 21:00 and 05:00. The resulting data set had 156 predictor variables from the 311 requests, the crime score, a crime x volume interaction term, and the two target variables--one binary, one continuous--for a total size of 160 features.

## Modeling
### Association Rule Mining
Apriori algorithm is used to generate association rules for:
<ol>
    <li>The entire dataset</li> 
    <li>Requests from High income areas</li>
    <li>Requests from Mid income areas</li>
    <li>Requests from Low income areas</li>
    <li>Requests from Mid & High Crime index areas</li>
    <li>Requests based on Neighborhood Profile Area</li>
</ol>

### Clustering
We decided to take all the numerical data in our 311 Calls dataset and see if we could cluster based on the these numbers. We were also able to include a categorical variable that relates if a neighborhood is a redline area or not. For us to be able to use a categorical variable we used the K-Prototypes clustering package in python. Once the clustering was done, we transfered the cluster ID back to the original data frame for mapping. Below are the results.
![image](https://user-images.githubusercontent.com/70532006/145243907-401b0e30-8db8-4cab-aab5-0709390febe6.png)


### Predictive Modeling
With such a large computed feature set, we expected and indeed observed a significant degree of multicollinearity amongst the predictors, which would have confounded our attempts at inference based on individual predictors' influences and significance. After attempting both LASSO regression and Recursive Feature Elimination with Cross-Validation, neither method removed enough variables to reduce the multicollinearity problem to a manageable level. Our solution was then a forward-pass stepwise selection regression to select the top 30 variables, from which we dropped an additional 3 with variance inflation factors > 10. 

With the resulting predictor set of 27 variables, we ran an ElasticNet *Linear Regression* to control for the remaining multicollinearity, while also allowing additional shrinkage of less important parameters by the L1-norm penalty. The target variable for this linear regression was a neighborhood's median household income.

With the same predictor set, we also ran an ElasticNet *Logistic Regression* to predict the binary target variable of whether the neighborhood was historically red-lined.

## Evaluation
### Association Rule Mining
Association Rules generated for each of these categories are:
<ol>
    <li>The entire dataset: <br/>
    NON_RECYCLABLE ITEMS and RECYCLABLE ITEMS are the highest in the frequest item sets which aligns with our data because these two are the highest reported type of 311 service requests. The next highest are CART, TRANSPORTATION, HNS HEALTH AND SANITATION, MISSED SERVICE, GARBAGE and RECYCLING. All these are likely to occur in the same neighborhood profile area along with NON_RECYCLABLE ITEMS and RECYCLABLE ITEMS within a time-span of a month. A few instances of 311 DOCUMENT is likely to occur with NON_RECYCLABLE ITEMS and RECYCLABLE ITEMS. <br/>
    We observe that if HNS HEALTH AND SANITATION, RECYCLING or YARD WASTE requests are raised, chances of GARBAGE, MISSED SERVICE, CART or RECYCLABLE ITEMS are likely to also be raised in a particular neighborhood profile area during a monthly timeframe. We also see a lot of association rules related to VIOLATIONS requests raised alond with some of the above mentioned types of requests.</li> 
    <li>Requests from High income areas: <br/>
    Similar to assocition rules for the entire dataset, we see that if NON_RECYCLABLE ITEMS and HNS HEALTH AND SANITATION requests are raised, chances of RECYCLABLE ITEMS request are likely to also be raised. Also, requests on TIRES have chances of raising NON_RECYCLABLE ITEMS in a particular neighborhood profile area during a monthly timeframe.</li>
    <li>Requests from Mid income areas: <br/>
    We see that if GARBAGE, YARD WASTE, RECYCLING requests are raised, chances of MISSED SERVICE, RECYCLABLE ITEMS, CART, TRANSPORTATION are likely to also be raised in a particular neighborhood profile area during a monthly timeframe. We also see the above requests in combination with NON_RECYCLABLE ITEMS and RECYCLABLE ITEMS requests. One difference we observe is, we don't see VIOLATIONS requests as part of the association rules for mid income group
    </li>
    <li>Requests from Low income areas: <br/>
    We see that if HNS HEALTH AND SANITATION, RECYCLING requests are raised, chances of RECYCLABLE ITEMS, NON_RECYCLABLE ITEMS, TRANSPORTATION are likely to also be raised in a particular neighborhood profile area during a monthly timeframe. We also see the above requests in combination with CART, GARBAGE and MISSED SERVICE requests. We observe that association rules for low income group have some combinations of VIOLATIONS requests.</li>
    <li>Requests from Mid & High Crime index areas: <br/>
    We see DEAD ANIMAL COLLECTION, 311 DOCUMENT, TRANSPORTATION and MISSED SERVICE to be common rules such that if any of these are raised, chances of RECYCLABLE ITEMS, NON_RECYCLABLE ITEMS, GARBAGE, CART, RECYCLING, HNS HEALTH AND SANITATION are likely to also be raised in a particular neighborhood profile area during a monthly timeframe. In contrast to low income datasets, we do not see VIOLATIONS requests in combinations of association rules, which makes us question if Crime and Violation requests are related to the crime index or low income areas.</li>
    <li>Requests based on Neighborhood Profile Area: <br/>
    Association rules show that certain request types tend to occur together during the same month in the neighborhood profile areas 3, 371, 378, 392, 393, 385</li>
![image](https://user-images.githubusercontent.com/64916499/145267317-deab8902-13c2-45a1-8262-bf2cbd1437ee.png)

</ol>

### Predictive Modeling

#### Linear Regression
The linear model showed that after correcting for multicollinearity in the predictors, 11 variables pertaining to 311 requests are highly significant features in predicting median income level of their respective neighborhoods. The model can account for roughly 26% of the variability in median income, and can do so with an average error of 22%, or approximately $14,000.
##### Linear Regression Validation Results:
![image](https://user-images.githubusercontent.com/78170609/145232562-e4a23bdd-5a41-44f3-b663-48d804463499.png)

![image](https://user-images.githubusercontent.com/78170609/145232320-cf6dd071-ff12-4f80-9ce1-c87e31041a92.png)

##### Inferences
Some of the observed effects are summarized below:

CART_VOL: Number of requests in the CART category per 1,000 people in one month
- From an increase by 1 request per thousand citizens in the average monthly volume of CART requests for a given neighborhood, we can expect with 95% confidence that our model's prediction of that neighborhood's median income will be between $4,908 and $5,621 lower. That is, poorer neighborhoods appear to have a higher per capita volume of CART-related requests.

RECYCLABLE ITEMS_VOL: Number of recyclable items requests per 1,000 people in one month
- From an increase by 1 request per thousand citizens in the average monthly volume of RECYCLABLE ITEMS requests for a given neighborhood, we can expect with 95% confidence that our model's prediction of that neighborhood's median income will be between $7,072 and $7,626 lower. Likewise, poorer neighborhoods appear to have a higher volume of recycling requests.

NON_RECYCLABLE ITEMS_VOL: Number of non-recyclable items requests per 1,000 people in one month
- From an increase by 1 request per thousand citizens in the average monthly volume of NON_RECYCLABLE ITEMS requests for a given neighborhood, we can expect with 95% confidence that our model's prediction of that neighborhood's median income will be between $1,769 and $1,979 higher. In this case, richer neighborhoods appear to have a higher volume of non-recyclable requests.

#### Logistic Regression
The model shows that 14 variables pertaining to 311 requests are highly significant features in predicting whether a neighborhood was historically a red-lined district, which it can predict with 76% accuracy.
##### Logistic Regression Validation Results:
![image](https://user-images.githubusercontent.com/78170609/145233477-d747dd7b-1652-46de-91b0-51236b270345.png)

![image](https://user-images.githubusercontent.com/78170609/145233726-e88bb664-9625-421a-9b68-e295f07d63cb.png)

##### Inferences
Some of the observed effects are summarized below:

CART_VOL: Number of requests in the CART category per 1,000 people in one month
- From an increase of 1 request per thousand citizens in the average monthly volume of CART requests for a given neighborhood, we can expect with 95% confidence that the odds of that neighborhood being labeled a historically redlined district will increase by a factor between 5.8 and 6.5. That is, higher per capita volume of CART-related requests is positively associated with historically redlined districts. The accuracy of the model's assigned label of course remains only 76%.

RECYCLABLE ITEMS_VOL: Number of recyclable items requests per 1,000 people in one month
- From an increase by 1 request per thousand citizens in the average monthly volume of RECYCLABLE ITEMS requests for a given neighborhood, we can expect with 95% confidence that the odds of that neighborhood being labeled a historically redlined district will decrease by a factor between 0.46 and 0.43 -- roughly a 55% reduction in the odds. That is, higher per capita volume of RECYCLABLE ITEM-related requests is negatively associated with historically redlined districts. The accuracy of the model's assigned label of course remains only 76%.

NON_RECYCLABLE ITEMS_VOL: Number of non-recyclable items requests per 1,000 people in one month
- From an increase by 1 request per thousand citizens in the average monthly volume of NON_RECYCLABLE ITEMS requests for a given neighborhood, we can expect with 95% confidence that the odds of that neighborhood being labeled a historically redlined district will increase by a factor between 3.37 and 3.46. That is, higher per capita volume of NON-RECYCLABLE ITEM-related requests is positively associated with historically redlined districts. The accuracy of the model's assigned label of course remains only 76%.

## Results
The predictive models indicate statistically significant relationships between 311 request behavior and median income as well as 311 request behavior and historical red-lining. The linear model explains only 26% of the total variance in median income with an average error of 22%, and the logistic model accurately predicts red-lined districts only 76% of the time. The predictive capabilities of both models are meager at best, but the results are sufficient to confirm the hypothesis behind our research question for this project:

**Neighborhoods within the city of Charlotte exhibit different 311 service request profiles, which appear to correlate with statistical significance to the neighborhood's median income, and to historical red-lining.**

Disclaimer: This project has neither investigated, inferred, nor implied any causal links relating to 311 service requests, income, and/or historical red-lining. The results above indicate a correlation which needs to be explored much further and in a more controlled manner in greater detail before any causality should be inferred.

## Future Work
### Possible future work may include:
<ol>
  <li>Improve existing model fit</li>
  <li>Explore correlations between weather and 311 service requests</li>
  <li>Predict high call volume days or periods to alleviate workload and wait time concerns</li>
  <li>Explore correlations between location and specific service requests' response times</li>
</ol>
