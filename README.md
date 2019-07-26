---
Author: Ping Zhou  
Email: nuistzhou@gmail.com  
Date: July 12th 2019

---
# Directory layout  
This is a rough overview of the top-level directory structure of the repo:


+&nbsp; - data   
|&nbsp; &nbsp; &nbsp; &nbsp;|   
|&nbsp; &nbsp; &nbsp; &nbsp;+- Whole_Foods_locations.csv    # Input food locations file.   
|&nbsp; &nbsp; &nbsp; &nbsp;|   
|&nbsp; &nbsp; &nbsp; &nbsp;+- Zip_MedianListingPrice_TopTier.csv    # Input Median Price List.  
|  
|&nbsp; &nbsp; - environment.yml   # Conda environment file.   
|  
|&nbsp; &nbsp; - main.ipynb   # The main Jupyter Notebook file.    
|  
|&nbsp; &nbsp; - output   # Folder where the enriched_property.csv lies.   

# Prerequisite

## Conda environment
Conda is a great choice for complex system
set-ups, as they often appear in big data and scientific use-cases including
the geo-sciences. To install build the data engineering pipeline in a fresh environment, I would therefore recommend to use
conda.

If have no Conda installed yet, please follow the installation instructions online, visit their [website](https://docs.conda.io/projects/conda/en/latest/user-guide/install/index.html).

After have Conda installed, create the conda environment and activate it with following commands:   

```bash
$ conda env create -f environment.yml    
$ conda activate realestate_scrapper
```

Run the Jupyter notebook:  

```bash
$ jupyter notebook
```
## Database   
My personal Postgres database running on my own VPS was used for this project. Use your own if possible.

# Summary
## Approach
* Web Scraping    
The Request library was used to retrieve html sources, which afterwards  
were parsed by the Beautiful Soup library. 
After the initial analysis of different property pages on the Curbed, it is noticed that those 
property pages have inconsistent structures, which made it difficult to be scraped, therefore, the 
scraping strategy was set to scrape whatever that we defined structured, while ignoring those unstructured ones.Generally, following are steps scraping properties' various attributes:   


    1. Url: from the Curbed "House of the day" page, each section has an url redirecting to the corresponding 
    property.  
    2. Property_id: the curbed property id in middle of each property's url was used to be the identifier in our table  
    3. Location: only at City-State level right now. By searching the keyword 'Location' from all P tags.       
    4. Price: By searching for keyword 'Price' from all P tags.
    5. Featured_date: parsed from the property unique url.
    6. Latitude, Longitude: applied the HERE geo location service API, using the City, State information, which means
     city center's geo coordinates were used instead of the property's exact ones.
     
    
    
* Enrich the property table with Median Listing Price 
    1. Load the csv file into a data frame and remove unnecessary months columns  
    2. Create a temporary table by joining the median price table into the property table, calculate median price, 3 
    months rolling median price and 6 months rolling median price. A property might be matched with several median 
    price records because the City + State combination was used to do the joining in last step, the average aggregation 
    was performed on those matches to get an unique result for each property.  
    3. Join the results from last step back to the property table to get the enriched result.
    
* Enrich the property table with the Food Location data set

    1. Create geometry for records from both properties and food locations in order to do the spatial analysis.
    2. Apply PostGIS spatial analysis function to count number of food locations within a near distance per property. 
    Here a quick research was did about how people perceive 'near' by driving since it is common for people go 
    grocery by car, 10-15 mins driving time was used and normally it is around 25 KM.
    
    
    
## Assumption  
Since the long-term web scarping really rely on an consistent and static web page structure, here we assumes
that the Curbed website stays static. However, when fails, it requires time-consuming maintenance. 

## Limitations and Discussions

* It is very important to have a reliable and consistent data source. But after the initial analysis of 
the 
Curbed website, it is believed that this website doesn't has a proper structure, it is advised to find new source 
instead. 

* Some properties failed to match any median price since the city and state combinations were not completely matched 
from both tables
. A better 
solution could be use the available median price from a neighbouring region instead for those properties. 

* Nearby food locations counting: it could be more intuitive and precise to apply the driving time isoline to 
calculate the number of foods locations within a certain driving time from the property, instead of using 
the linear distance, but of course, the exact geo location of the property is needed for this.

* Parsing the property's address for geo coding the exact geo coordinates: several regular expression were 
experimented, for instance *(\d{1,}) [a-zA-Z0-9\s]+(\.)? [a-zA-Z]+(\,)?*, however, only some of the properties match 
this pattern, thus a better regular expression is required. Alternatively, applying some open source NLP (Natual Language Processing) address 
extraction solutions could also help with a relative high accuracy, due to limited time reason, this can be researched further in the future.

* Enriching the Property table with Median Listing Price: although the Python way was used to implement this step, 
another way should be using SQL completely, for example, the function **crosstab** could be helpful to achieve this 
goal by producing a "pivot table".

* The pipeline stills has a small probelm: somehow during aggregation for the 3 month rolling and 6 month rolling, extreme invalid value poped up, have no time to figure it out at the moment. For details, see the final exported enriched property csv file.
