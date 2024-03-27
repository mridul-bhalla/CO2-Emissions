# CO2-Emissions around the globe


## Table of Contents

- [Project Overview](#project-overview)
- [Data Sources](#data-sources)
- [Tools Used](#tools-used)
- [Data Collection](#data-collection)
- [Data Preparation and Cleaning](#data-preparation-and-cleaning)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [Data Analysis](#data-analysis)
- [Results](#results)
- [Limitations](#limitations)
- [References](#references)



## Project Overview

This data analysis project aims to understand the CO2 emission trends worlwide considering both Production and Consumption based carbon emissions. 
- Production based emissions are calculated by adding all the emissions that are produced within that country.
- While Consumption based emissions are calculated by adding up all the emissions associated with the goods and services consumed within that country.

By analysing various aspects of the emissions data, we seek to understand the differences in trends of carbon emissions between developed and developing countries, and gain a deeper understanding of whether and how emissions are impacted by population growth and growth in GDP. 

A major part of this project involves webscraping data and handling missing values.



## Data Sources

1. Production & Consumption based emissions data:
   - [List of Countries by C02 emissions per capita](https://en.wikipedia.org/wiki/List_of_countries_by_carbon_dioxide_emissions_per_capita)
   - The data was scraped using BeautifulSoup from the above wikipedia page. This served as the primary data for analysis.
  
     
2. Population by Countries over the years
   - The csv file related to population data was downloaded from [World Bank Open Data](https://data.worldbank.org/indicator/SP.POP.TOTL)
   - Data File: 'year_wise_population_by_country.csv' file
     
3. GDP by Countries over the years
   - The excel file related to GDP data was downloaded from [wits.worldbank.org](https://wits.worldbank.org/CountryProfile/en/country/by-country/startyear/ltst/endyear/ltst/indicator/NY-GDP-MKTP-CD)
   - Data File: 'year_wise_gdp_by_country.xlsx' file
     
4. Consumption based CO2 per capita
   - The csv file related to Consumption based data was downloaded from [OurWorldInData.org](https://ourworldindata.org/consumption-based-co2)
   - Data File: 'consumption-co2-per-capita.csv'
  
5. Countries by Continents
   - Data Source: [List of Countries by Continents wikipedia page](https://simple.wikipedia.org/wiki/List_of_countries_by_continents)
   - Data File: 'Continents.xlsx'

  

## Tools Used

- Python
- Libraries include:
  1. BeautifulSoup - for Webscraping
  2. Pandas & Numpy - for Data Cleaning & Analysis
  3. Sklearn - for KNN and Iterative Imputation
  4. Matplotlib, Seaborn, Plotly, hvplot - for Data Analysis & Visualisations 



## Data Collection

In the inital data collection phase, we performed the following tasks:
1. Webscraped data using BeautifulSoup from before mentioned data sources
   - Code snippet of web scraping using BeautifulSoup:
     ```python
     
     # importing the required library
     from bs4 import BeautifulSoup as bs

     # Downloading contents of the webpage
     response = requests.get("https://en.wikipedia.org/wiki/List_of_countries_by_carbon_dioxide_emissions_per_capita")
     webpage = response.text

     # Creating BeautifulSoup Object
     soup = bs(webpage, "html.parser")

     # Grabbing the table
     h2 = soup.select_one("span#Notes").parent
     table = h2.find_previous_sibling('table')
     print(table)

     # grabbing row data
     rows = table.select(selector="tbody tr")
     row_content = []
     for each_tr in rows:
       all_tds = each_tr.find_all('td')
       row = [str(each_td.get_text()).strip() for each_td in all_tds]
       row_content.append(row)
     print(row_content)

     # making a df - using row content, setting column names
     co2_df = pd.DataFrame(row_content, columns=['Country/Territory', 'Production (1990)', 'Consumption (1990)',
                                       'Production (2005)', 'Consumption (2005)', 'Production (2017)',
                                        'Consumption (2017)', 'Production (2020)', 'Consumption (2020)',
                                       'Production (2021)', 'Consumption (2021)', 'C-P (2020)',
                                       'Percentage diff in C-P (2020)', 'Net (2020)'])

     # dropping first 5 empty rows and resetting the index
     co2_df.drop(index=co2_df.index[:4], inplace=True)
     co2_df.reset_index(inplace=True, drop=True)
     co2_df.head()
     
     ```
2. Data loading & inspection
3. Merging dataframes to create a proper dataset.



## Data Preparation and Cleaning

1. Handling missing values
   - We perfomed three kinds of imputations to deal with missing values and compared them to work with best data possible.
     - Median Imputation
     - KNN Imputatation
     - Iterative Imputation
   - A comparison using KDE Plot
  <img width="1121" alt="KDE PLOT" src="https://github.com/mridul-bhalla/CO2-Emissions/assets/158173545/108ee783-626c-4980-9cd2-8c24fe2301b5">


2. Creating new columns/features
3. Data Cleaning & Formatting



## Exploratory Data Analysis

EDA involved exploring the emissions data to answer key questions, such as:
- What are the top 5 countries with highest production & consumption based emissions?
- What is the Trend of Production/Consumption based Co2 Emission in Each Continent Over the Years?
- Number of emissions exporting & importing countries over the time
- Is there any relationship between CO2 emissions per capita and GDP per capita?
- Is there any relationship between CO2 emissions per capita and population?



## Data Analysis

A snippet of code of creating an interactive line graph that shows country wise emissions.
- It allows toggling between Production and Consumption based emissions
- Allows choosing a particular country using a dropdown.
  ```python
     
  # importing the required libraries
  import holoviews as hv
  from holoviews import dim, opts
  hv.extension('bokeh', 'matplotlib')

  # Make dataframe pipeline interactive
  idf = emissions.interactive()

  # Setting Multiselect for country
  my_country_list = []
  for each_country in emissions["Country Name"].unique():
    my_country_list.append(each_country)
  multi_select = pn.widgets.MultiSelect(name='Country', value=['Afghanistan'],
    options=[each_country for each_country in my_country_list], size=8)

  # Radio buttons for switching between Production based emissions and Consumption based emissions
  yaxis_co2 = pn.widgets.RadioButtonGroup(
    name='Y axis', 
    options=['Production', 'Consumption'], 
    button_type='danger')

  # creating data pipleine for this graph
  co2_pipeline = (
    idf[
        (idf["Country Name"].isin(multi_select))
    ]
    .groupby(["Country Name", "Year"])[yaxis_co2].mean()
    .to_frame()
    .reset_index()
    .sort_values(by="Year")
    .reset_index(drop=True)
  )

  # Plotting the graph
  co2_plot = co2_pipeline.hvplot(x = "Year", by="Country Name", y=yaxis_co2, line_width=2, title="CO2 emissions by Country")
  co2_plot
     
   ```



## Results

The analysis results are summarised as follows:
- Qatar has been the highest production based CO2 emitter over the years, and it ranks 2nd in consumption based emissions after UAE.
- Continents & Countries around the world showed a dip in emissions in 2020, but in 2021, emissions started to increase. Covid-19 can be one of the reasons for the same.
- Moreover, a positive relationship has been observed between emissions per capita and gdp per capita.
- Rich countries/Developed countries (with high GDP) are high polluters and have a low population level.
- While, the more populated countries are middle income countries and they emit at a near average level, some are high emitters as well



## Limitations

The data related to Consumption based emissions was incomplete and had many missing values. Although the dataset was imputed using best possible methods, still it doesn't assures 100% accuracy. Also, at places, certain assumptions have been taken into consideration for analysing the data. One such assumption is that countries having gdp_per_capita >= 20,000$ are considered as Developed. Also there might be a few outliers in the data. 



## References

1. [Understanding Production vs Consumption Based emissions](https://www.youtube.com/watch?v=fHUn6BFFRYQ)
2. [Stack Overflow](https://stack.com)
3. [KNN Imputation](https://medium.com/@bhanupsingh484/handling-missing-data-with-knn-imputer-927d49b09015)
4. [Iterative Imputation](https://towardsdatascience.com/iterative-imputation-with-scikit-learn-8f3eb22b1a38)
5. [Matplotlib Documentation](https://matplotlib.org/)
6. [Seaborn Documentation](https://seaborn.pydata.org/)
7. [Plotly Documentation](https://plotly.com/python/)
8. [Hvplot Documentation](https://hvplot.holoviz.org/)







