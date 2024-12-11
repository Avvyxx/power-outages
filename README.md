## **Table of Contents**
1. [Introduction](#introduction) <br>
    a. [Power Outage Raw Dataset](#data_desc) <br>
    b. [Necessary Columns](#col_desc)
2. [Data Cleaning and Exploratory Data Analysis](#cleaning) <br>
    a. [Data Cleaning](#data_clean) <br>
    b. [Univariate Analysis](#uni_analysis) <br>
    c. [Bivariate Analysis](#bi_analysis) <br>
    d. [Interesting Aggregates](#aggr)
3. [Assessment of Missingness](#assess_missingness) <br>
    a. [NMAR Analysis](#NMAR) <br>
    b. [Missingness Dependency](#missingness) <br>

## Introduction <a name="introduction"></a>

Power outages are a common occurrence that impacts communities across the United States, with lasting effects on both residential and commercial properties. The causes of power outages may vary between weather conditions, infrastructure issues, or even intentional human intervention. And, as climate change worsens in recent years, winter storms and summer heat waves have raised concerns about reoccurring widespread power disruptions. Thus, the goal of this project is to analyze the impact of winter or summer weather on the prevalence of power outages in the U.S., specifically exploring whether a certain season is worse than the other. By analyzing this data, we aim to identify trends and inform better preparedness strategies for cities enduring winter outages throughout the United States.

The research question I aim to answer is:
> ***Are major power outages equally likely to occur between summer and winter?***


### Power Outage Raw Dataset <a name="data_desc"></a>

To tackle my research question, I will be drawing from Purdue's dataset on [Major Power Outage Risks in the U.S.](https://engineering.purdue.edu/LASCI/research-data/outages) This dataset provides a detailed record of **1,534 rows/observations** of power outages across the United States, spanning multiple years and containing **56 columns** of different variables. The first five observations from the raw data can be seen below:

Key attributes include the start and restoration times of outages, geographical information (e.g., state and region), and the causes of outages (e.g., severe weather, intentional attacks). In addition, it incorporates demographic and economic data such as population density, land usage, and measures of regional economic performance, which add valuable context to the analysis.

### Necessary Columns <a name="col_desc"></a>

For the first half of this project, focus will be placed on specific columns most relevant to exploring the relationship between seasonal patterns and power outage frequencies in detail. Specifically, we shall analyze the following columns from our *cleaned* dataset as described by [Data on major power outage events in the continental U.S.](https://www.sciencedirect.com/science/article/pii/S2352340918307182). Justifications for their usage are also described below:

* `YEAR`                      - The year when the power outage event occurred.
                                <br> Certain years may have had unique trends or weather events that may influence the outcome of our analysis.

* `MONTH`                     - The month when the outage event occurred.
                                <br> Indicative of the season (i.e. December–February for winter), which helps identify seasonal patterns in outages.

* `OUTAGE.DURATION`            - Duration of the outage events (in minutes).
                                <br> Indicative of the severity of the outage and the grid’s ability to recover.

* `POSTAL.CODE`                - State abbreviation in the U.S. where the outage event occurred. 
                                <br> Certain states may influence the outcome of our analysis
                                     (i.e. California has fewer winter storms than most other states)
  
* `NERC.REGION`               - The North American Electric Reliability Corporation regions involved in the outage event.
                                <br> Each NERC region has different grid management practices and policies, distinct climates, and infrastructure
                                resilience strategies that can impact outage dynamics.
  
* `CLIMATE.REGION`            - U.S. Climate regions, as specified by the National Centers for Environmental Information.
                                <br> Certain climate regions are more or less prone to seasonal storms and extreme weather events, providing insight 
                                into regional vulnerabilities.

* `ANOMALY.LEVEL`             - Represents the El Niño/La Niña index, which influences seasonal climate patterns.
                                <br> These anomalies can affect weather patterns such as precipitation, storms, and temperature extremes, potentially
                                increasing outage risks.

* `CLIMATE.CATEGORY`          - Represents "Warm," "Cold," or "Normal" climate episodes based on Oceanic Niño Index.
                                <br> Identifies broader seasonal climate trends that might correlate with outages
                                (e.g., a "Cold" episode may bring more snowstorms, more indicative of winter).
  
* `CAUSE.CATEGORY`            - General categories of events causing the power outages.
                                <br> Useful to determine if the power outage was weather-related or not during our analyses.
  
* `HURRICANE.NAMES`           - If applicable, the hurricane name associated with the outage. **NaN if no hurricane present**.
                                <br> Deeper understanding regarding if outages were more attributed to the season or external weather events.
 
* `SEASON`                    - Main focus of our research. Either 'Winter', 'Spring', 'Summer', or 'Fall'

 Now, let's get into the data cleaning!

## Data Cleaning & Exploratory Data Analysis <a name="cleaning"></a>

### Data Cleaning <a name="data_clean">

To clean the data, we must first address the following columns: `'OUTAGE.START.DATE'`, `'OUTAGE.START.TIME'`, `'OUTAGE.RESTORATION.DATE'`, `'OUTAGE.RESTORATION.TIME'`:

|          | OUTAGE.START.DATE   | OUTAGE.START.TIME   | OUTAGE.RESTORATION.DATE   | OUTAGE.RESTORATION.TIME   |
|---------:|:--------------------|:--------------------|:--------------------------|:--------------------------|
|    **0** | 2011-07-01 00:00:00 | 17:00:00            | 2011-07-03 00:00:00       | 20:00:00                  |
|    **1** | 2014-05-11 00:00:00 | 18:38:00            | 2014-05-11 00:00:00       | 18:39:00                  |
|    **2** | 2010-10-26 00:00:00 | 20:00:00            | 2010-10-28 00:00:00       | 22:00:00                  |
|    **3** | 2012-06-19 00:00:00 | 04:30:00            | 2012-06-20 00:00:00       | 23:00:00                  |
|    **4** | 2015-07-18 00:00:00 | 02:00:00            | 2015-07-19 00:00:00       | 07:00:00                  |
|  **...** | ...                 | ...                 | ...                       | ...                       |
| **1529** | 2011-12-06 00:00:00 | 08:00:00            | 2011-12-06 00:00:00       | 20:00:00                  |
| **1530** | NaT                 | nan                 | NaT                       | nan                       |
| **1531** | 2009-08-29 00:00:00 | 22:54:00            | 2009-08-29 00:00:00       | 23:53:00                  |
| **1532** | 2009-08-29 00:00:00 | 11:00:00            | 2009-08-29 00:00:00       | 14:01:00                  |
| **1533** | NaT                 | nan                 | NaT                       | nan                       |

Notice how for these columns, our date-time objects have been split into two columns. We can aggregate the data within the `.DATE` and `.TIME` to fully represent a date-time object like so, reducing our columns and limiting redundancy:

|          | OUTAGE.START                  | OUTAGE.RESTORATION            |
|---------:|:------------------------------|:------------------------------|
|    **0** | 2011-07-01 17:00:00           | 2011-07-03 20:00:00           |
|    **1** | 2014-05-11 18:38:00           | 2014-05-11 18:39:00           |
|    **2** | 2010-10-26 20:00:00           | 2010-10-28 22:00:00           |
|    **3** | 2012-06-19 04:30:00           | 2012-06-20 23:00:00           |
|    **4** | 2015-07-18 02:00:00           | 2015-07-19 07:00:00           |
|  **...** | ...                           | ...                           |
| **1529** | 2011-12-06 08:00:00           | 2011-12-06 20:00:00           |
| **1530** | NaT                           | NaT                           |
| **1531** | 2009-08-29 22:54:00	       | 2009-08-29 23:53:00           |
| **1532** | 2009-08-29 11:00:00	       | 2009-08-29 14:01:00           |
| **1533** | NaT                           | NaT                           |

Now that all our column data types are set, let's address another concern: repeated data. Notice how some columns contain the same type of data:
* `U.S._STATE` and `POSTAL.CODE` both pertain to the state of the recorded power outage.
* `OBS` is the unique identifier of the **obs**ervation, similar to our index.

We can drop redundant data to reduce the number of columns even further.

And now a peak at the first 5 rows of my cleaned dataset looks like this:

|        |   YEAR |   MONTH | POSTAL.CODE   | NERC.REGION   | CLIMATE.REGION     |   ANOMALY.LEVEL | CLIMATE.CATEGORY   | CAUSE.CATEGORY     | CAUSE.CATEGORY.DETAIL   |   HURRICANE.NAMES |   OUTAGE.DURATION |   DEMAND.LOSS.MW |   CUSTOMERS.AFFECTED |   RES.PRICE |   COM.PRICE |   IND.PRICE |   TOTAL.PRICE |   RES.SALES |   COM.SALES |   IND.SALES |   TOTAL.SALES |   RES.PERCEN |   COM.PERCEN |   IND.PERCEN |   RES.CUSTOMERS |   COM.CUSTOMERS |   IND.CUSTOMERS |   TOTAL.CUSTOMERS |   RES.CUST.PCT |   COM.CUST.PCT |   IND.CUST.PCT |   PC.REALGSP.STATE |   PC.REALGSP.USA |   PC.REALGSP.REL |   PC.REALGSP.CHANGE |   UTIL.REALGSP |   TOTAL.REALGSP |   UTIL.CONTRI |   PI.UTIL.OFUSA |   POPULATION |   POPPCT_URBAN |   POPPCT_UC |   POPDEN_URBAN |   POPDEN_UC |   POPDEN_RURAL |   AREAPCT_URBAN |   AREAPCT_UC |   PCT_LAND |   PCT_WATER_TOT |   PCT_WATER_INLAND | OUTAGE.START        | OUTAGE.RESTORATION   |
|---:|-------:|--------:|:--------------|:--------------|:-------------------|----------------:|:-------------------|:-------------------|:------------------------|------------------:|------------------:|-----------------:|---------------------:|------------:|------------:|------------:|--------------:|------------:|------------:|------------:|--------------:|-------------:|-------------:|-------------:|----------------:|----------------:|----------------:|------------------:|---------------:|---------------:|---------------:|-------------------:|-----------------:|-----------------:|--------------------:|---------------:|----------------:|--------------:|----------------:|-------------:|---------------:|------------:|---------------:|------------:|---------------:|----------------:|-------------:|-----------:|----------------:|-------------------:|:--------------------|:---------------------|
|  **0** |   2011 |       7 | MN            | MRO           | East North Central |            -0.3 | normal             | severe weather     | nan                     |               nan |              3060 |              nan |                70000 |       11.6  |        9.18 |        6.81 |          9.28 | 2.33292e+06 | 2.11477e+06 | 2.11329e+06 |   6.56252e+06 |      35.5491 |      32.225  |      32.2024 |         2308736 |          276286 |           10673 |           2595696 |        88.9448 |        10.644  |       0.411181 |              51268 |            47586 |          1.07738 |                 1.6 |           4802 |          274182 |       1.75139 |             2.2 |      5348119 |          73.27 |       15.28 |           2279 |      1700.5 |           18.2 |            2.14 |          0.6 |    91.5927 |         8.40733 |            5.47874 | 2011-07-01 17:00:00 | 2011-07-03 20:00:00  |
|  **1** |   2014 |       5 | MN            | MRO           | East North Central |            -0.1 | normal             | intentional attack | vandalism               |               nan |                 1 |              nan |                  nan |       12.12 |        9.71 |        6.49 |          9.28 | 1.58699e+06 | 1.80776e+06 | 1.88793e+06 |   5.28423e+06 |      30.0325 |      34.2104 |      35.7276 |         2345860 |          284978 |            9898 |           2640737 |        88.8335 |        10.7916 |       0.37482  |              53499 |            49091 |          1.08979 |                 1.9 |           5226 |          291955 |       1.79    |             2.2 |      5457125 |          73.27 |       15.28 |           2279 |      1700.5 |           18.2 |            2.14 |          0.6 |    91.5927 |         8.40733 |            5.47874 | 2014-05-11 18:38:00 | 2014-05-11 18:39:00  |
|  **2** |   2010 |      10 | MN            | MRO           | East North Central |            -1.5 | cold               | severe weather     | heavy wind              |               nan |              3000 |              nan |                70000 |       10.87 |        8.19 |        6.07 |          8.15 | 1.46729e+06 | 1.80168e+06 | 1.9513e+06  |   5.22212e+06 |      28.0977 |      34.501  |      37.366  |         2300291 |          276463 |           10150 |           2586905 |        88.9206 |        10.687  |       0.392361 |              50447 |            47287 |          1.06683 |                 2.7 |           4571 |          267895 |       1.70627 |             2.1 |      5310903 |          73.27 |       15.28 |           2279 |      1700.5 |           18.2 |            2.14 |          0.6 |    91.5927 |         8.40733 |            5.47874 | 2010-10-26 20:00:00 | 2010-10-28 22:00:00  |
|  **3** |   2012 |       6 | MN            | MRO           | East North Central |            -0.1 | normal             | severe weather     | thunderstorm            |               nan |              2550 |              nan |                68200 |       11.79 |        9.25 |        6.71 |          9.19 | 1.85152e+06 | 1.94117e+06 | 1.99303e+06 |   5.78706e+06 |      31.9941 |      33.5433 |      34.4393 |         2317336 |          278466 |           11010 |           2606813 |        88.8954 |        10.6822 |       0.422355 |              51598 |            48156 |          1.07148 |                 0.6 |           5364 |          277627 |       1.93209 |             2.2 |      5380443 |          73.27 |       15.28 |           2279 |      1700.5 |           18.2 |            2.14 |          0.6 |    91.5927 |         8.40733 |            5.47874 | 2012-06-19 04:30:00 | 2012-06-20 23:00:00  |
|  **4** |   2015 |       7 | MN            | MRO           | East North Central |             1.2 | warm               | severe weather     | nan                     |               nan |              1740 |              250 |               250000 |       13.07 |       10.16 |        7.74 |         10.43 | 2.02888e+06 | 2.16161e+06 | 1.77794e+06 |   5.97034e+06 |      33.9826 |      36.2059 |      29.7795 |         2374674 |          289044 |            9812 |           2673531 |        88.8216 |        10.8113 |       0.367005 |              54431 |            49844 |          1.09203 |                 1.7 |           4873 |          292023 |       1.6687  |             2.2 |      5489594 |          73.27 |       15.28 |           2279 |      1700.5 |           18.2 |            2.14 |          0.6 |    91.5927 |         8.40733 |            5.47874 | 2015-07-18 02:00:00 | 2015-07-19 07:00:00  |

Note that some columns are aggregations other columns (i.e. duration is derived from start to restoration time), but these columns will be kept in mind for potential future analyses.

Next, let's separate the columns needed for our current analysis. Since month is essential to our research question (for seasons), we will also be dropping rows where `MONTH` is NaN before adding in our `SEASON` column as described earlier:

#### A Note on NaNs

All NaNs in `CLIMATE.REGION` are in Hawaii, since Hawaii being an island in the far Pacific does not have a designated [climate region](https://www.ncei.noaa.gov/access/monitoring/reference-maps/us-climate-regions) within the United States. For this reason, we will impute the values with "Tropical Island" to indicate its unique region.

As previously discussed, missing values in `HURRICANE.NAME` indicate that no hurricane occured during those events, making these NaNs meaningful by design. Similarly, missingness in `OUTAGE.DURATION` arises when outage start and restoration times were not reported in the corresponding columns. Given the nature of these variables:
* We choose to retain the NaNs in `HURRICANE.NAME`, as their presence directly indicates a non-hurricane event and does not require imputation.
* While missing values in `OUTAGE.DURATION` can potentially limit analyses related to outage severity, this column is not central to our research question. Instead, we will retain it as well as the current NaN values as means to explore potential trends during the exploratory data analysis stage.

And now, this subset of data is ready for analysis!

|          |   YEAR |   MONTH |   OUTAGE.DURATION | POSTAL.CODE   | NERC.REGION   | CLIMATE.REGION     |   ANOMALY.LEVEL | CLIMATE.CATEGORY   | CAUSE.CATEGORY                |   HURRICANE.NAMES | SEASON   |
|---------:|-------:|--------:|------------------:|:--------------|:--------------|:-------------------|----------------:|:-------------------|:------------------------------|------------------:|:---------|
|    **0** |   2011 |       7 |              3060 | MN            | MRO           | East North Central |            -0.3 | normal             | severe weather                |               nan | Summer   |
|    **1** |   2014 |       5 |                 1 | MN            | MRO           | East North Central |            -0.1 | normal             | intentional attack            |               nan | Spring   |
|    **2** |   2010 |      10 |              3000 | MN            | MRO           | East North Central |            -1.5 | cold               | severe weather                |               nan | Fall     |
|    **3** |   2012 |       6 |              2550 | MN            | MRO           | East North Central |            -0.1 | normal             | severe weather                |               nan | Summer   |
|    **4** |   2015 |       7 |              1740 | MN            | MRO           | East North Central |             1.2 | warm               | severe weather                |               nan | Summer   |
|  **...** |    ... |     ... |               ... | ...           | ...           | ...                |             ... | ...                | ...                           | ...               |    ... |
| **1527** |   2016 |       3 |               nan | ID            | WECC          | Northwest          |             1.6 | warm               | intentional attack            |               nan | Spring   |
| **1528** |   2016 |       7 |               220 | ID            | WECC          | Northwest          |            -0.3 | normal             | system operability disruption |               nan | Summer   |
| **1529** |   2011 |      12 |               720 | ND            | MRO           | West North Central |            -0.9 | cold               | public appeal                 |               nan | Winter   |
| **1531** |   2009 |       8 |                59 | SD            | RFC           | West North Central |             0.5 | warm               | islanding                     |               nan | Summer   |
| **1532** |   2009 |       8 |               181 | SD            | MRO           | West North Central |             0.5 | warm               | islanding                     |               nan | Summer   |

### Univariate Analysis <a name="uni_analysis">

While our analyses will be focusing on on the `SEASON` of our data, let's explore the distribution of major outage causes:

<iframe
  src="figs/univar1_causes.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

As expected, severe weather dominates major outage events with 763 outages. This figure highlights the large role weather-related factors, such as storms, hurricanes, and extreme temperatures, play in causing power disruptions. Severe weather can often be attributed to the season as well, with summers and winters being the most severe in the US in most cases. However, intentional attacks come unexpectedly second in this distribution at 418 outages—no other categories fall close. We'll take a closer look into this later.

Next in our univariate analysis is a box plot depicting the distribution of outage duration (in minutes) from over the years. Recall that there are some NaNs in this data, but let's see if we can still make our some significant trends:

Without excluding outliers, the distribution of the data is heavily right skewed. Some outliers go as far as over 100 thousand minutes! Before we investigate this, let's look at our duration distribution *without* those outliers.

<iframe
  src="figs/univar2_1_durations.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Given that these events are *major* outages, the right skewed distribution aligns with our expectations—in severe situations, an outage can last from hours to even days. Thus, despite the NaNs, our `OUTAGE.DURATION` data is ample enough to reflect potential trends.

#### Outlier Investigation

|    |   YEAR |   MONTH |   OUTAGE.DURATION | POSTAL.CODE   | NERC.REGION   | CLIMATE.REGION     |   ANOMALY.LEVEL | CLIMATE.CATEGORY   | CAUSE.CATEGORY        |   HURRICANE.NAMES | SEASON   |
|---:|-------:|--------:|------------------:|:--------------|:--------------|:-------------------|----------------:|:-------------------|:----------------------|------------------:|:---------|
| 53 |   2014 |       1 |            108653 | WI            | RFC           | East North Central |            -0.5 | cold               | fuel supply emergency |               nan | Winter   |

And it looks like this extreme outlier of ours was a result of a fuel supply emergency (lack of coal) during the winter season!

### Bivariate Analysis <a name="bi_analysis">

Let's now take a look at the frequency of major power outages for each state over the years by investigating the value counts of `POSTAL.CODE` per year. The figure can then provide a general understanding of the geospatial distribution of the data.

<iframe
  src="figs/bivar1_states.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

States that are more population dense have noticibly more power outages that rural states. This highlights an important consideration: confounding factors. Discrepancies in infrastructure, population, etc. play huge roles in differences between states. We must also remember that the dataset reports *major* power outages, and thus larger states (e.g. California's larger population density) will result in larger impacts on even smaller-scale outages when compared to smaller states like Alaska's sparser population.

Next, let's take a deep dive into outages a *monthly* basis with a focus on severe weather:

<iframe
  src="figs/bivar2_weather.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Interestingly enough, summer has more presence than winter. And as expected, spring and fall months tend to have less occurences of major power outages.

### Interesting Aggregates <a name="aggr"></a>

Now, as mentioned earlier, `"intentional attack"` was unexpectedly high as a cause for major power outages. Let's take a closer look on our outage categories and how they fair against one another in the states with the most outages:

| POSTAL.CODE   |   equipment failure |   fuel supply emergency |   intentional attack |   islanding |   public appeal |   severe weather |   system operability disruption |
|:--------------|--------------------:|------------------------:|---------------------:|------------:|----------------:|-----------------:|--------------------------------:|
| CA            |                  21 |                      17 |                   24 |          28 |               9 |               70 |                              41 |
| MI            |                   3 |                       0 |                    4 |           1 |               1 |               83 |                               3 |
| NY            |                   2 |                      12 |                   13 |           0 |               4 |               33 |                               7 |
| TX            |                   5 |                       6 |                   13 |           0 |              17 |               65 |                              20 |
| WA            |                   1 |                       1 |                   64 |           5 |               1 |               24 |                               1 |

Interestingly enough, Washington's outages are dominated by intentional attacks, and a quick Google search reveals that Washington state is a common victim of grid attacks. Thus, it is important to consider that severe weather and the season, despite its dominance in most

## Assessment of Missingness <a name="assess_missingness"></a>

Taking a step back, let's analyze the missing data in our **raw** dataset. Of course, some missingness already outlined to us (i.e. `HURRICANE.NAMES` is only able to be provided if a hurricane is present). Or, as we analyzed earlier, Hawaii does not have a recognized Climate Region thus causing missingness in the corresponding column as well. But for most columns with NaNs, it seems uncertain whether there is a pattern to this behavior, or whether these data are missing at random.

### NMAR Analysis <a name="NMAR"></a>

Based on domain knowledge and the data generation process, I believe that the missingness in the `IND.SALES` ("Electricity consumption in the industrial sector, in megawatt-hours") column is NMAR (Not Missing at Random). Several industrial sectors may choose not to report their electricity consumption due to concerns over competitive advantage or public reputation. For example, large industries may want to keep their energy usage private to prevent revealing sensitive information about their production volumes or operational inefficiencies that contribute to the larger conversation of climate change. With increasing attention on sustainability, some industries may prefer to withhold data to avoid scrutiny over energy inefficiency.


In fact, taking a deeper dive into our data, we observe that missing data in any of the regional electricity consumption column all columns ***always*** results in missing data in all columns (`'RES.PRICE'`, `'COM.PRICE'`, `'IND.PRICE'`, `'TOTAL.PRICE'`, `'RES.SALES'`, `'COM.SALES'`, `'IND.SALES'`, `'TOTAL.SALES'`, '`RES.PERCEN'`, `'COM.PERCEN'`, and `'IND.PERCEN'`). Because of this pattern, the missing data is likely due to intentional withholding of information, further supporting the NMAR assumption.

|          |   RES.PRICE |   COM.PRICE |   IND.PRICE |   TOTAL.PRICE |   RES.SALES |   COM.SALES |   IND.SALES |   TOTAL.SALES |   RES.PERCEN |   COM.PERCEN |   IND.PERCEN |
|---------:|------------:|------------:|------------:|--------------:|------------:|------------:|------------:|--------------:|-------------:|-------------:|-------------:|
|  **339** |         nan |         nan |         nan |           nan |         nan |         nan |         nan |           nan |          nan |          nan |          nan |
| **1533** |         nan |         nan |         nan |           nan |         nan |         nan |         nan |           nan |          nan |          nan |          nan |
|  **887** |         nan |         nan |         nan |           nan |         nan |         nan |         nan |           nan |          nan |          nan |          nan |
|  **365** |         nan |         nan |         nan |           nan |         nan |         nan |         nan |           nan |          nan |          nan |          nan |
|  **766** |         nan |         nan |         nan |           nan |         nan |         nan |         nan |           nan |          nan |          nan |          nan |
|  **239** |         nan |         nan |         nan |           nan |         nan |         nan |         nan |           nan |          nan |          nan |          nan |
| **1318** |         nan |         nan |         nan |           nan |         nan |         nan |         nan |           nan |          nan |          nan |          nan |
| **1506** |         nan |         nan |         nan |           nan |         nan |         nan |         nan |           nan |          nan |          nan |          nan |
| **1530** |         nan |         nan |         nan |           nan |         nan |         nan |         nan |           nan |          nan |          nan |          nan |
|  **900** |         nan |         nan |         nan |           nan |         nan |         nan |         nan |           nan |          nan |          nan |          nan |
| **1520** |         nan |         nan |         nan |           nan |         nan |         nan |         nan |           nan |          nan |          nan |          nan |
| **1528** |         nan |         nan |         nan |           nan |         nan |         nan |         nan |           nan |          nan |          nan |          nan |
|  **103** |         nan |         nan |         nan |           nan |         nan |         nan |         nan |           nan |          nan |          nan |          nan |
|  **743** |         nan |         nan |         nan |           nan |         nan |         nan |         nan |           nan |          nan |          nan |          nan |
|  **762** |         nan |         nan |         nan |           nan |         nan |         nan |         nan |           nan |          nan |          nan |          nan |
| **1419** |         nan |         nan |         nan |           nan |         nan |         nan |         nan |           nan |          nan |          nan |          nan |
|  **827** |         nan |         nan |         nan |           nan |         nan |         nan |         nan |           nan |          nan |          nan |          nan |
|  **605** |         nan |         nan |         nan |           nan |         nan |         nan |         nan |           nan |          nan |          nan |          nan |
|   **36** |         nan |         nan |         nan |           nan |         nan |         nan |         nan |           nan |          nan |          nan |          nan |
|  **171** |         nan |         nan |         nan |           nan |         nan |         nan |         nan |           nan |          nan |          nan |          nan |
|  **218** |         nan |         nan |         nan |           nan |         nan |         nan |         nan |           nan |          nan |          nan |          nan |
|  **417** |         nan |         nan |         nan |           nan |         nan |         nan |         nan |           nan |          nan |          nan |          nan |

### Missingness Dependency <a name="missingness"></a>

Now, how about other columns that appear to have sporadic missing values? Let's see if we can determine whether the missingness for some columns is dependent on one and independent of another, therefore missing at random (MAR).

#### Is missingness of `OUTAGE.DURATION` dependent on `CAUSE.CATEGORY`?

`OUTAGE.DURATION` as we know is an aggregate statistic, but we can use the column to understand when *timestamps* weren't reported in general. Perhaps in severe power outage causes, the durations were unable to be reported. Let's check this out!

<iframe
  src="figs/missingness_cause_categories.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Looks like values are especially missing during fuel supply, which makes sense since nonessential processes like logging time stamps may be shut down prior to a major outage event to conserve fuel. Similarly, equipment failure has a high prevalence of missing values over non-missing, likely because the methods of collecting logs may have had a misfunction prior to the total major outage. Let's investigate further with a hypothesis test.

For this test, we will operate on a significance level of 0.01, or 1%.:

* Null Hypothesis: The distribution of `CAUSE.CATEGORY` when `OUTAGE.DURATION` data is missing is the same as the distribution of `CAUSE.CATEGORY` when `OUTAGE.DURATION` is not missing.
* Alternative Hypothesis: The distribution of `CAUSE.CATEGORY` when `OUTAGE.DURATION` data is missing is NOT the same as the distribution of `CAUSE.CATEGORY` when `OUTAGE.DURATION` is not missing.

<iframe
  src="figs/missingness_cause_categories_dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Based on our observed TVD of 0.252, we observe a p-value of 0.0, and so we reject the null.

The distribution of `CAUSE.CATEGORY` when `OUTAGE.DURATION` data is missing is NOT the same as the distribution of `CAUSE.CATEGORY` when `OUTAGE.DURATION` is not missing, and therefore the missingness of `OUTAGE.DURATION` may be dependent on `CAUSE.CATEGORY`.

#### Is missingness of `OUTAGE.DURATION` dependent on `MONTH`?

To make this analysis clearer, let's map `MONTH` to the respective month labels (ie. 1 to January)

<iframe
  src="figs/missingness_months.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

At first glance, it is hard to tell if there is a relationship between the two. Let's investigate further.

For this test, we will operate on a significance level of 0.01, or 1%.:

* Null Hypothesis: The distribution of `MONTH.MAPPED` when `OUTAGE.DURATION` data is missing is the same as the distribution of `MONTH.MAPPED` when `OUTAGE.DURATION` is not missing.
* Alternative Hypothesis: The distribution of `MONTH.MAPPED` when `OUTAGE.DURATION` data is missing is NOT the same as the distribution of `MONTH.MAPPED` when `OUTAGE.DURATION` is not missing.

<iframe
  src="figs/missingness_months_dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Based on our observed TVD of 0.2227, we observe a p-value of 0.099, and so we FAIL to reject the null.

There is evidence to support that the distribution of `MONTHS` when `OUTAGE.DURATION` data is missing is the same as the distribution of `MONTHS` when `OUTAGE.DURATION` is not missing, and therefore the missingness of `OUTAGE.DURATION` may be not dependdent on `MONTHS`.

## Hypothesis Testing

I will be testing whether there is an equal probability that an outage will occur in Summer as in Winter.

Null Hypothesis: The number of outages in Summer and Winter are from the same distribution (i.e., there is no significant difference between them).

Alternative Hypothesis: The number of outages in Summer and Winter come from different distributions (i.e., there is a significant difference between them).

Now, we will perform a permutation test to gather evidence.

If our null hypothesis holds true, then the rate of Summer and Winter outages is the same throughout a year, and shuffling the values of the `SEASON` column should have little effect on the value of our statistic. Thus, we will repeatedly shuffle the `SEASON` column and record the difference between the average number of winter and summer per year. Then, we will compare the values of the test statistics we obtain to our observed statistic, and see whether our observed statistic is significant.

<iframe
  src="figs/simulated_test_stats.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

<iframe
  src="figs/hyptest_fig.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

## Baseline Model

***Prediction Problem: Predicting the Outage Cause***

In this second half of our report, I shall aim to predict the **cause**, `CAUSE.CATEGORY`, of a major power outage based on several influencing factors. Note that we will be basing our model on data we can know at the START of an outage (i.e. we cannot use restoration time). The cause of an outage can vary from severe weather events (e.g., storms) to equipment failures or intentional attacks. By accurately predicting the cause, utilities can develop better strategies for preventing, responding to, and managing outages.

- **Problem Type**: Classifier (RandomForest)
- **Prediction Variable**: `CAUSE.CATEGORY`
- **Evaluation Metric**: F1 Score (because not all categories equally represented, as seen in our EDA)

### Features

To predict the `CAUSE.CATEGORY` of an outage, we will first train a baseline model using columns most highly attributed to the cause of an outage.

- `POSTAL.CODE`: Certain states may be more prone to certain major power outages (i.e. Washington and intentional attacks)
- `SEASON`: The season in which the outage occurs (Spring, Summer, Fall, Winter), as certain seasons (e.g., winter storms, summer heatwaves) might be associated with certain outages.
- `HURRICANE.PRESENCE`: If a hurricane is involved, the outages are likely related to severe weather.
- `CLIMATE.REGION`: The geographical climate area where the outage occurs, as some climates may be more prone severe weather outages or other categories.

## Final Model

My final model incorporates `OUTAGE.START`, `NERC.REGION`, `TOTAL.CUSTOMERS`, and `POPULATION` into the predictive process. I added `OUTAGE.START` becase the time of day may influence our analysis, `NERC.REGION` because certain regions may be more likely to be affected by a specific cause, and `TOTAL.CUSTOMERS` and `POPULATION` because there may be a correlation between the amount of customers and intentional attacks.
