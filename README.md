# COVID-19 Impact on Entrepreneurship

By Elizabeth Mullinax

## Questions Asked and why they matter 

1. Did the COVID19 shutdown lead more people to start their own businesses?
2. Were people more likely to start a business during the mandate? 

I strongly believe that the COVID19 pandemic kickstarted a cultural shift to make America more entrepreneurial. Hypothetically, as people lost trust in employers to provide their necessary income due to the layoffs during shutdowns they began  to turn to themselves to create their own income. With that in mind, this analysis is primarily going to focus on business applications submitted by non-corporations (individuals, partnerships, etc. ""small groups"") during the COVID19 Stay-at-home mandates. 

## Datasets Used

- [Monthly Nationwide Unemployment Rate from Bureau of Labor Statistics](https://data.bls.gov/timeseries/LNS14000000) 
    The dataset is a monthly time series published by the BLS, showing the nationwide U.S. unemployment rate (age 16+, seasonally adjusted) from January of 2015 - September of 2025. It features 13 columns (year and months) and 11 rows (header and years.) 

- [Monthly Business Formation Statistics from the Census](https://www.census.gov/econ/bfs/current/index.html) 
    The Monthly Business Formation Statistics comes from the U.S. Census Bureau. The dataset counts the number of businesses that have just applied for an EIN during a given month and year by state, industry, and series type. The series type column identifies which specific statistic the row reports whether its business applications, business applications by corporations, actual formations within 4 or 8 quarters, projected formations, or average durations from application to formation. It has a total of over 35,000 rows across 17 columns.

- [Weekly Business Formation Statistics from the Census](https://www.census.gov/econ/bfs/current/index.html) 
    The Weekly Business Formation Statistics also comes from the U.S. Census Bureau.  The dataset counts the number of businesses that have just applied for an EIN during a given week and year by state, industry, and application type. Unlike the monthly dataset, the weekly dataset does *not* look at business already formed. It has a total of seven columns and 52276 rows. 

  
- [Annual Estimates of the Resident Population for the United States, Regions, States, District of Columbia and Puerto Rico: April 1, 2020 to July 1, 2024](https://www.census.gov/data/tables/time-series/demo/popest/2020s-state-total.html) 
    This dataset is published by the U.S. Census Bureau and provides annual United States, state and region population estimates from April 1st 2020 - July 1st 2024. The dataset featured 62 rows including states, US totals, other regions, as well as multiple header rows. It featured seven columns, one for the geographic area, one for an estimate as of April 2021, then five rows of yearly estimates as of July 1st going from 2020 - 2024. 
  
- [U.S. State, Territorial, and County Stay-At-Home Orders: March 15-May 5 by County by Day](https://data.cdc.gov/Policy-Surveillance/U-S-State-Territorial-and-County-Stay-At-Home-Orde/qz3x-mf9n/about_data)
    This dataset was published by the Centers for Disease Control and Prevention (CDC). It documents state, territorial, and county stay‑at‑home orders during COVID‑19 (March 15–May 5, 2020). Each row represents a jurisdiction (state, territory, or county) on a given day, and columns capture identifiers (state, county, FIPS codes), dates, and the type/status of stay‑at‑home order. The dataset’s columns capture who issued the order, where it applied, when it was active, and what type of restrictions were in place. 


## Data Manipulations

For each Data Visualization, it was filtered to show only non-corporation business applications. States with unclear or no stay-at-home mandates were also filtered out. Given that the Business Formation Statistics dataset doesn't record on a daily basis and stay-at-home mandates often start in the middle of the week, there's a small gap in data on the exact number of businesses formed at the beginning of the stay at home mandate and end of the week that the stay at home mandate begun. A daily average of business application was calculated so an approximate number of businesses formed could be found to fill this gap. Also given that the 

### Calculated Fields

***Non-Corporate Business Applications, for the Monthly Business Formation Statistics Dataset***

```
{ FIXED [State] : 
  SUM(IF [Series] = "BA_BA" THEN [Business Application Amt] END) - 
  SUM(IF [Series] = "BA_CBA" THEN [Business Application Amt] END) }
```

This subracts the number of corporate business applications, labeled BA_CBA in the Monthly Business Formation Statistics Dataset from the number of total business applications, labeled BA_BA. 


***Non-Corporate Business Applications, for the Weekly Business Formation Statistics Dataset***

``` 
 { FIXED [Start Date], [End Date], [State] : 
  SUM(IF [Series] = "BA_NSA" THEN [Business Application Amt] END) - 
  SUM(IF [Series] = "CBA_NSA" THEN [Business Application Amt] END) }
``` 

This subracts the number of corporate business applications, labeled CBA_NSA from the number of total business applications, BA_NSA wihtin a given week which then feeds into the:

***Daily Average of Non-Corporate Business Applications***

``` [Non-Corporate Business Applications]/7 ```

Since the number of business applications is given on a weekly basis, we divide by 7 to get the daily average of business applications made within each week. 

***Daily Average of Non-Corporate Business Applications Per Capita***

``` [Daily Average of Non-Corporate Business Applications]/[Population Value] ```

Given that states with higher population values will show a higher daily average of non-corporate business applications, we must divide the Daily Average of Non-Corporate Business Applications by population value to lessen the variance. 

***COVID19 Change in Daily Average of Non-Corporate Business Applications Per Capita (Mandate States)***

{ FIXED [State] : 
  AVG(IF [During Stay-at-Home Mandate] = TRUE 
      THEN [Daily Average of Non-Corporate Business Applications Per Capita] END) 
}
-
{ FIXED [State] : 
  AVG([Daily Average of Non-Corporate Business Applications Per Capita])
}

***Mandatory Stay at Home Order Lengths***

```
{ FIXED [County Name], [State] : 
  DATEDIFF('day', 
    MIN(IF [Stay at Home Order Recommendation] = "Mandatory for all individuals" 
           AND [Effective date] != "NA"
        THEN DATEPARSE('M/d/yyyy', [Effective date]) END),
    MAX(IF [Stay at Home Order Recommendation] = "Mandatory for all individuals" 
           AND [Expiration date] != "NA"
        THEN DATEPARSE('M/d/yyyy', [Expiration date]) END)
  )
}
```

How long was the mandatory stay at home order length by county? 

***During Stay at Home Mandate***

``` 
[Start Date] >= DATE([Effective date]) 
AND [End Date] <= DATE([Expiration date])
AND [Stay at Home Order Recommendation] = "Mandatory for all individuals"
``` 
Was the business application submitted during a mandatory stay at home mandate? 


## Analysis and Results 


### Unemployment Rate vs Number of Non-Corporate Businesses Applications, 2015 - 2024 

![Non-Corporation Business Applications VS Unemployment Rate.png](Non-Corporation Business Applications VS Unemployment Rate)

The COVID19 pandemic first shut down businesses in March of 2020 leading to an increase in Unemployment in April 2020. Following this, there was a spike of business applications in July of 2020. Given the time it takes to plan a business and process a business application, I believe its within reason to find a direct correlation between the covid19 shutdowns and people starting their own businesses. This number of people starting their own businesses long after the end of the covid19 pandemic is still much higher than the number of people starting their own businesses before the covid19 pandemic. 

### Change in Daily Average of Non-Corporation Business Applications Per Capita During Stay-at-Home Mandates

![Change in Daily Average of Non-Corporation Business Applications Per Capita During Stay-at-Home Mandates.png](Change in Daily Average of Non-Corporation Business Applications Per Capita During Stay-at-Home Mandates)

Contrary to my hypothesis, the number of business applications generally decreased during the COVID19 pandemic. 

## Known Variances 
Here are known potential causes of issues within the datasets
- The Business Formations are recorded by State and Week, not day or county. 
- The COVID19 Stay-at-Home Mandates dataset doesn't include information on expiration dates for stay-at-home mandates that were announced after May 5th. So the information we have on states or counties with very long stay-at-home mandates is sparse. 
- The IRS may have experienced delays in processing EIN applications during the COVID19 pandemic. 
- 