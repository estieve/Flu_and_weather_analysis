# Flu_and_weather_analysis

## Compiling the Data

To complete this project I had to collect and combine data from multiple sources. There are two primary pieces: 1) the data on flu infections, and 2) the weather data. For the weather data I accessed the API from the National Weather Service (https://www.ncdc.noaa.gov/cdo-web/). The code in my "Compile_Data" notebook was used to pull temperature and precipitation data for a span ranging from 2009-2019. 

### Weather Data
The NOAA api only allows for 365 day of data in a single pull with one attribute, to get around this I built a series of loops to get three attributes per year and then combine the 10 year span into a single dataset. I created the tool in such a way that I could repeat the process easily with different stations to create a compiled list for more than one location. Unfortunately there are both gaps in the datasets where no records are available for specific days, and some outliers where egregious values are present such as precipitation totals greater than 100 inches and temperatures higher than 120°. 

![WeatherMal_outliers](https://github.com/estieve/Flu_and_weather_analysis/blob/main/Images/WeatherMal_outliers.PNG)

These are things that I had to deal with in the weather dataset before I could move forward with adding the flu data. To do so, I removed the egregious values by making then Nan's and reseting exceptionally low values to a specified minimum (0 for precipitation and the location's lowest seasonal temperature for temp). The Nan's I then reset to a mean value since there was a wide distribution of Nan data and filling forward created high plateau peaks of data. The final step involved prepping the data for merging with the weekly flu data, to do this I converted our dataframe of daily records into a new dataframe that calculated weekly averages for our attributes. I then converted the precipitation data to a weekly total because I felt that better represents the amount of precipitation for a given week.

### Flu Data
The influenza data was collected from two different sources, the WHO FluNet (https://www.who.int/influenza/gisrs_laboratory/flunet/en/)surveillance system and from the New York State Department of Health (https://health.data.ny.gov/Health/Influenza-Laboratory-Confirmed-Cases-By-County-Beg/jr8b-6gh6/data). These were extracted as csv files directly from the websites and I chose data for New York City, Luxembourg, and Malta. All of the associated files are in the repo as LUX_Flu, Mal_Flu, and NY_FLU. This data required less cleaning and primarily consisted of removing extraneous columns to get down to the core factors which were end date for the specified week and the count for number of positive flu cases.

### Flu and Weather
I created an output dataframe for each location that included the key weather and flu data joined on the date column. With that completed I then joined them together into a single dataframe titled "flu_weather". I exported this as a csv to use in my data analysis.

## EDA
