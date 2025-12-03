# Overview

In this project, I will demonstrate data preparation and aggregation methods, together with data analysis, on the example of wheat futures data. I will use Power BI charts, as well as Python and R visuals, for a more in-depth analysis. 
## Data was used from these sources: 
- WASDE Reports https://esmis.nal.usda.gov/publication/world-agricultural-supply-and-demand-estimates#release-items
- Commitments of Traders Reports https://www.cftc.gov/MarketReports/CommitmentsofTraders/index.htm
- US Export Sales https://apps.fas.usda.gov/export-sales/?ref=ag.hedder.com
- Weather data https://www.cpc.ncep.noaa.gov/products/analysis_monitoring/cdus/degree_days/
-pbix link  
  



# Goal & Data overview
Our **goal is to model factors that influence commercials' net positioning in futures and options, so we can see how extreme the positioning is compared to previous years**. This will give us insights about future price movements. The data sets that we will use are Commitment of Traders Reports, WASDE reports, Export Sales reports, OHLC data, and Cooling/Heating Degree days. 

<img width="1384" height="635" alt="1 1 Positioning" src="https://github.com/user-attachments/assets/06234c1c-f89e-4ccd-852c-fa1189e81279" />

<img width="1078" height="605" alt="1 2 Key Influencers" src="https://github.com/user-attachments/assets/5d6179c1-9971-40e4-b4f8-e4271273f49f" />

<img width="1078" height="605" alt="1 3 Month Decomposition" src="https://github.com/user-attachments/assets/9f6dfab8-6a08-4bad-8f88-6d687b8417e5" />

We will be using z-scores with a rolling average with a look-back window of five years. This will allow us to directly compare deviations from averages rather than raw values. Before we proceed with transformations, we will take a quick glance at correlation tables that will give us a hint of potential relationships.(1. [Correlation](https://github.com/DmitryBatyuk-1/wheat/blob/468b8d3e687c7765afc288261ac0b195a3307974/1.4%20Correlation%20PY) ) 



# Data Preparation and structure
First, we aggregate WASDE reports into one ( 2. [Aggregation Script](https://github.com/DmitryBatyuk-1/wheat/blob/468b8d3e687c7765afc288261ac0b195a3307974/2.%20Aggregation%20Script%20PY)); Here are the links two sample data before: [wasde09.24.xls](https://github.com/DmitryBatyuk-1/wheat/blob/468b8d3e687c7765afc288261ac0b195a3307974/wasde0924.xls) Since our data has different granularity and was collected on different days of the week, we need to prepare the data for Python and R so that the regression scripts run properly.

We create a calendar table and merge a copy of it with each weekly fact table. As an output, we get a daily fact table that has some null values, which we fill down to complete the data. (3. Weekly to Daily transformation) Now we have data that is ready to be linked. We use a star schema to link our calendar with multiple fact tables. The only thing that unites them is the date. 

# Variable preparation
We have already measured correlations of raw variables with the Net Commercial position. One of the problems of using raw data is the potential non-stationarity of the time series. We will transform variables of interest into z-scores that will be calculated taking a five-year rolling average. After the transformation, the relationship still exists. We will proceed with our analysis. (1.4 correlation summary z-score) 

# Modeling
We will run OLS, including the Newey-West estimator with five lags, so our standard errors are more reliable.(5. OLS) The model will fit our values, and the residuals that deviate more than one sigma (adjustable threshold) will indicate unusually high or low positioning, which will serve as a potential indicator that the Net Commercial position is about to reach its peak. 

Our standard errors are most likely overly “optimistic” due to potential autocorrelation; however, R-squared and coefficients are still meaningful. 


# Results
If we plot our residuals, we can see points in time where the positioning was very extreme. The bigger the deviation, the stronger the signal. (5. Residuals and Signal)
