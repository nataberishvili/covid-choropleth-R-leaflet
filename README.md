# covid-choropleth-R-leaflet
Interactive Maps with R leaflet Covid-19

Interactive data visualization enhances exploratory data analysis and is a great way to engage with both technical and non-technical audiences. R's leaflet package is a powerful tool to create visually compelling interactive maps. In this post I will show how to create a choropleth map with leaflet. Choropleth maps show the level of variability within a region, using color.  
I assume the reader has some experience with R programming and familiarity with the tidyverse library. Let's install the leaflet package and call the libraries.
install.packages("leaflet")     
library(tidyverse)  
library(leaflet)  
Step 1. Download Data and Prepare for Mapping.  


Today we will build a choropleth map using data for the 2019 Novel Coronavirus published by Johns Hopkins University Center for Systems Science and Engineering. To make it easy to follow through the steps, we will work with state-level data. The same approach can be applied to any geographic dataset.
You can find the dataset and metadata [here].    
library(readr)  
urlfile="https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_daily_reports_us/09-18-2020.csv"  
covid <-read_csv(url(urlfile))  
We can see 58 rows and 18 columns of data, including entities like  
Cruise Ships and American territories. We will use the 50 States and District of Columbia to simplify the mapping.  
covid <- covid[-c(3, 10, 14, 15, 40, 45, 53),]  
When selecting and preparing a dataset for choropleth, we need a population-adjusted metric. Today we will use Covid-19 incident rates per 100,000 persons.
Examples of other population-adjusted data appropriate for choropleth are:  
Percentage change in cases by County.  
Number of incidents per 100.000 by State.  
Case fatality ratios by County.
Rate increase in Covid-19 Cases by State.

Raw numbers, unadjusted for population, are generally unsuitable for choropleth.


Step 2. Shapefiles for Mapping
Shapefiles are standardized geometric shapes for geographic regions. The R tigris package by Kyle Walker allows users to download and use TIGER/Line shapefiles from the US Census Bureau, or we can download directly from the USCB.
Let's install the package, call the library and get the shapefiles for the States and DC.
install.packages('tigris')
library(tigris)
states <- states(cb = TRUE)
Next, we use the geo_join function to merge our dataset and shapefiles, creating a spatial data frame.
data <- geo_join(states, covid, "NAME", "Province_State", how="inner")


Step 3. Getting Started with Base USA Map
To create a USA base map we will call the three main functions
Create a map widget by calling leaflet().
provides a base map and different theme options withaddProviderTiles(), Find all options on this link.
setView() - centers the map, specifying latitude, longitude, and zoom level. Google, can identify latitude and longitude of any location. Center point coordinates for the US map are (-73.935242, 40.730610).

map <- leaflet() %>%
 addProviderTiles("ldfldf") %>%
 setView(-73.935242, 40.730610, zoom = 8)
map
Step #3. Base map

Step 4. Color Palette
When creating a choropleth map, choosing the right colors and number of classes is mission critical towards efficiently telling our story. Since we are using a population-adjusted metric, our default approach involves picking a single color and adjusting saturation proportionately to our data. But there are many other approaches, and we should consider the data distribution in selecting our palette:
Do we have extreme outliers in the data?
What are facts we need to draw attention to?

pal <- colorNumeric(palette = "Blues", domain=data$Incident_Rate)
Here I specified the fill color as "Blues" based on incident rate. So the darker blues will show the higher numbers (incident rates) and the lighter color the smaller numbers.
Step 5. Create html markers
Now let's create a description/text appear on hover. We going to show the State name and incident rate. It is required information when hovered on particular region.
data$popup_h <- paste("State:", data$NAME, "/", "Mortality Rate:", (round(data$Mortality_Rate, 4)))
Step 6. Add Polygons
Here we actually layer our data/shapefile on our base map. Specifying the coloring option, and passing html marker - popup_h that we created in step 5. So while hover over different states it will show different values.
map3 <- leaflet() %>%
  addProviderTiles("CartoDB.Positron") %>% 
  setView(-98.5795, 39.8282, zoom = 4) %>% 
  addPolygons(data = data, 
              fillColor = ~pal(data$Incident_Rate),
              fillOpacity = 0.7, 
              weight = 2, 
              opacity = 1,
              color = "white",
              dashArray = "3",
              smoothFactor = 0.2, 
              label = data$popup_h)
map3
Step #6

Step 7. Add legends 
For the final step we will add legend. Legend is the key to explain what the different shades mean. We will specify 4 parameters 
color palette used for mapping (Step #4)
values that we used for mapping
position of the legend - right bottom corner 
title of the legend

Here is the final map
map3 <- leaflet() %>%
  addProviderTiles("CartoDB.Positron") %>% 
  setView(-98.5795, 39.8282, zoom = 4) %>% 
  addPolygons(data = data, 
              fillColor = ~pal(data$Incident_Rate),
              fillOpacity = 0.7, 
              weight = 2, 
              opacity = 1,
              color = "white",
              dashArray = "3",
              smoothFactor = 0.2, 
              label = data$popup_h) %>%
   addLegend(pal = pal, 
            values = data$Incident_Rate, 
            position = "bottomright", 
            title = "Covid Rate - 09/20/2020")
map3


Step #7. Final MapHere is our final map! 
You can further play with it and customize the map, according to your data and preferences!
Thank you for reading!
