# Overview

In this project, I will demonstrate data preparation and aggregation methods, together with data analysis, on the example of wheat futures data. I will use Power BI charts, as well as Python and R visuals, for a more in-depth analysis. [PowerBI Report Link](https://app.powerbi.com/reportEmbed?reportId=5e7c090c-2fc4-4bc7-be03-0c3bc24106c3&autoAuth=true&ctid=80f21a0b-c2f5-4593-a920-391b9cb2e666)
## Data was used from these sources: 
- WASDE Reports https://esmis.nal.usda.gov/publication/world-agricultural-supply-and-demand-estimates#release-items
- Commitments of Traders Reports https://www.cftc.gov/MarketReports/CommitmentsofTraders/index.htm
- US Export Sales https://apps.fas.usda.gov/export-sales/?ref=ag.hedder.com
- Weather data https://www.cpc.ncep.noaa.gov/products/analysis_monitoring/cdus/degree_days/

  



# Goal & Data overview
Our **goal is to model factors that influence commercials' net positioning in futures and options, so we can see how extreme the positioning is compared to previous years**. This will give us insights about future price movements. The data sets that we will use are Commitment of Traders Reports, WASDE reports, Export Sales reports, OHLC data, and Cooling/Heating Degree days. 

<img width="1384" height="635" alt="1 1 Positioning" src="https://github.com/user-attachments/assets/06234c1c-f89e-4ccd-852c-fa1189e81279" />

<img width="1078" height="605" alt="1 2 Key Influencers" src="https://github.com/user-attachments/assets/5d6179c1-9971-40e4-b4f8-e4271273f49f" />

<img width="1078" height="605" alt="1 3 Month Decomposition" src="https://github.com/user-attachments/assets/9f6dfab8-6a08-4bad-8f88-6d687b8417e5" />

Our data exhibits significant month-to-month seasonality. A more meaningful comparison involves examining the positioning of Net commercials in the same month across previous years. To conduct this analysis, we will employ z-scores calculated using a rolling average with a five-year look-back window. This will allow us to directly compare deviations from averages rather than raw values. Before we proceed with transformations, we will take a quick glance at correlation tables that will give us a hint of potential relationships.(1. [Correlation](https://github.com/DmitryBatyuk-1/wheat/blob/468b8d3e687c7765afc288261ac0b195a3307974/1.4%20Correlation%20PY) ) 

<img width="969" height="545" alt="1 4 correlation" src="https://github.com/user-attachments/assets/d95a3476-bcff-4a3b-85b6-91a02342d8d7" />
<img width="755" height="623" alt="1 4 correlation WASDE US" src="https://github.com/user-attachments/assets/658228aa-7e58-4256-8377-15a9b5da6c13" />
<img width="683" height="565" alt="1 4 correlation Summary" src="https://github.com/user-attachments/assets/97b8a35b-6268-402c-b1f8-507939fb7279" />



# Data Preparation and structure
First, we aggregate WASDE reports into one ( 2. [Aggregation Script](https://github.com/DmitryBatyuk-1/wheat/blob/468b8d3e687c7765afc288261ac0b195a3307974/2.%20Aggregation%20Script%20PY)); Here are the links to sample data before: [wasde09.24.xls](https://github.com/DmitryBatyuk-1/wheat/blob/468b8d3e687c7765afc288261ac0b195a3307974/wasde0924.xls) [wasde10.24.xls](https://github.com/DmitryBatyuk-1/wheat/blob/68782046b00d1c10d404c1dd73ab1747a05162f5/wasde1024.xls) and after: [WASDE_Compiled.xlsx](https://github.com/DmitryBatyuk-1/wheat/blob/68782046b00d1c10d404c1dd73ab1747a05162f5/WASDE_Compiled.xlsx)  Since our data has different granularity and was collected on different days of the week, we need to prepare the data for Python and R so that the regression scripts run properly.

We create a calendar table and merge a copy of it with each weekly fact table. As an output, we get a daily fact table that has some null values, which we fill down to complete the data. (3. Weekly to Daily transformation) Now we have data that is ready to be linked. We use a star schema to link our calendar with multiple fact tables. The only thing that unites them is the date. 
<img width="801" height="698" alt="3  Weekly to Daily transformation" src="https://github.com/user-attachments/assets/8ea37595-6114-44f0-ae0e-c43880d657aa" />

## Star Schema

<img width="1022" height="617" alt="Schema" src="https://github.com/user-attachments/assets/98cc6d15-435f-4450-88bc-4010e5ab9292" />


# Variable preparation
We have already measured correlations of raw variables with the Net Commercial position. One of the problems of using raw data is the potential non-stationarity of the time series. We will transform variables of interest into z-scores that will be calculated taking a five-year rolling average. ( 3. [WASDE](https://github.com/DmitryBatyuk-1/wheat/blob/589a5f6645c34dda07af8f169c29fae473510a94/3.%20WASDE%20DAX))

<img width="683" height="576" alt="1 4 correlation Summary Z score" src="https://github.com/user-attachments/assets/8a5668a8-24b6-43ab-8693-87598d830744" />

After the transformation, the relationship still exists. We will proceed with our analysis.

# Modeling
We will run OLS, including the Newey-West estimator with five lags, so our standard errors are more reliable.(4. [Model](https://github.com/DmitryBatyuk-1/wheat/blob/589a5f6645c34dda07af8f169c29fae473510a94/4.%20Model%20R)) 
<img width="696" height="381" alt="4  OLS" src="https://github.com/user-attachments/assets/4a77a8b3-c991-428b-95f0-a208cdbf7355" />

We can see that factors such as Crude Oil Spreads, the Trade Weighted US Dollar Index, Ending Stocks, and Heating Degree Days significantly influence the Commercial Net Position. Factors such as Accumulated Exports can be dropped from the model. By collecting and updating data on significant factors we can generate a weekly signal for trade execution. The model will fit our data, and residuals that deviate by more than one sigma (an adjustable threshold) will indicate unusually high or low positioning, which will serve as a potential indicator that the Net Commercial Position is about to reach its peak. Our standard errors are most likely overly "optimistic" due to potential autocorrelation; however, the R-squared value and coefficients are still meaningful.
<img width="1060" height="555" alt="4  Residuals" src="https://github.com/user-attachments/assets/f02bc6dd-f787-45bd-939d-8dc52fa6f3ad" />
<img width="1060" height="572" alt="4  Signal" src="https://github.com/user-attachments/assets/caa496ba-c031-4edb-83f1-fde0345c1ded" />



# Results
If we look at the Wheat Commercials Signal graph, we can see the residuals plotted against time. On the graph, we can see points in time where the deviation in our significant factors was very extreme (more than 1 sigma). These points represent our trading opportunities (mean reversion strategy). The greater the deviation, the stronger the signal.
