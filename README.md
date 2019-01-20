------------

# **TRIP PLANNER**

------------
One of the challenges folks who travel to various destinations face is finding a one stop shop for all their travel needs; i.e. places to see, restaurants to dine in, events to attend to, price ranges for all such activities etc...
Our team was tasked with the assignment of assisting a couple who plan to travel to San Diego with their travel plans with the main intent of alleviating the aforementioned challenges. As part of this task, we took on the responsibility of finding them resources on where to go, events to attend, must see attractions, restaurants they could easily find, in order to make their stay as pleasant and hassle free as possible. The couple were more interested in staying around the Mission Beach area; as that is where most of the tourist attraction sites are located.
## Finding Data
- The initial stage of our project began with finding the useful data, and as such we landed on three websites that we believed were useful:-
	1. [sandiego.org](http://sandiego.org "sandiego.org") - also known as San Diego Events - to gather events in and around San Diego, location of these events, price information and other details
	1.  [tripadvisor.com](http://tripadvisor.com "tripadvisor.com")  - to scrape best reviewed attraction sites in and around Mission Beach
	1.  [yelp.com](http://yelp.com "yelp.com") - to scrape restaurants and dining spots within 2 miles radius from Mission Beach
- once the team decided on what websites we plan to work on, we moved on to the discussion of methods and ways of scraping the needed information from the listed websites.

## Data Scraping, Cleanup & Analysis

**Scraping/Extract**
- To easilly scrape the needed data from the three sites, we used a free google chrome extension called webscraper.io (https://www.webscraper.io/). once the extension was downloaded and added to google chrome browser, the tool lets the user make selections of web content such as text, link, images, or buttons.
   - to scrape tripadvisor.com, we initially narrowed down our search criteria to only include*"things to do near Mission Beach"*for which a URL was generated as - https://www.tripadvisor.com/AttractionsNear-g60750-d104125-Mission_Beach-San_Diego_California.html. Using this URL, we used WebScraper to recursively select *Name, Address, Category, # of Reviews, and Cost* across all the pages of the search result. Once we completed our selections, the tool enabled us to scrape for these mentioned elements and save all in a CSV format.
   - We followed similar strategy for San Diego Events to store - *Event, Description, Neighborhood, Date and Cost*; and from Yelp.com to store - *Name, Address, Category, # of Reviews, Cost* .

**Cleanup/Transform**
- Inorder to cleanup our data, we used MS Excel and Tableau Prep; which helped us remove duplicates, extra spaces, and formatting numbers and texts.
- within our code, we also did some adjustments of our data; to include - adding "*formatted_cat*" field from SD_Events dataset by splitting the *"Category"*  column to remove the numbers:
	`SD_events_df['formatted_cat'] = SD_events_df.Category.map(lambda x: ' '.join(x.split(' ')[3:]))`
- we added a "*Category*" field to the yelp_df and populated this field with "*Food and Drink*" entry, so that we can have category values for all of our records; and also enable an easy filter capability once our database has been populated.
`yelp_df["Category"] = "Food & Drink"`
- once our three datasets have been combined using `combined_files = pd.concat([SD_events_df,trip_advisor_df, yelp_df], sort=True)`, we did further cleaning up to remove duplicates and convert datatypes to appropriate types.

**Load**
- we used MySQL to create a database -*"trip_planner_db"*that we would use to upload our data
- we used SQLAlchemy library to establish connection to our MySQL database; truncate all the records within the tables (to avoid duplicate entries everytime we execute the code) and load the cleaned data to three designated tables - *"attractions", "events" & "restaurants"*:
     - `connection_string = "root:Password@localhost/trip_planner_db?charset=utf8"engine = create_engine(f'mysql://{connection_string}')`
`engine.execute("TRUNCATE TABLE events;")`
`engine.execute("TRUNCATE TABLE attractions;")`
`engine.execute("TRUNCATE TABLE restaurants;")`
`SD_events_df.to_sql(name='events', con=engine, if_exists='append', index=False)`
`trip_advisor_df.to_sql(name='attractions', con=engine, if_exists='append', index=False)`
`yelp_df.to_sql(name='restaurants', con=engine, if_exists='append', index=False)`

### Analysis
- we used a free library -*"geocoder"* to generate the longitudes and latitudes of the addresses for our tripadvisor and yelp data so we can easily map the coordinates using Tableau Tableau Desktop.
   1. [![Trip Planner GoeMap.JPG](ETL-Project2/Resources/Trip Planner GoeMap.JPG "Trip Planner GoeMap.JPG")](http://github.com/cschurter/ETL-Project2/blob/master/Resources/Trip%20Planner%20GoeMap.JPG "Trip Planner GoeMap.JPG")
   1. [![Yelp_Mapping.JPG](ETL-Project2/Resources/Yelp_Mapping.JPG "Yelp_Mapping.JPG")](http://github.com/cschurter/ETL-Project2/blob/master/Resources/Yelp_Mapping.JPG "Yelp_Mapping.JPG")
- we created a historgam for SD events showing the price range, neighborhood and date informations.
    1. [![SD_Events.JPG](ETL-Project2/Resources/SD_Events.JPG "SD_Events.JPG")](http://github.com/cschurter/ETL-Project2/blob/master/Resources/SD_Events.JPG "SD_Events.JPG")

### List of Files Inside Repo needed to Recreate Project
- ETL-Project2.ipynb - jupyter notebook file containing all codes - needed to run code
- Resources - folder contiaining all needed source files and images
     - SD_Events.csv - extracted file for SD Events, needed for loading and processing file
	 - TripAdvisor.csv - extracted file for tripadvisor site, needed for loading and processing file
	 - Yelp_restaurants.csv - extracted file for yelp site, needed for loading and processing file
	 - SD_Events.JPG, Trip Planner GoeMap.JPG, Yelp_Mapping.JPG - image files for read me (not needed to recreate project).
	 - ETL_SQL_ Queries.txt - SQL statement to create DB and tables in My SQL (not needed to run the code in JupyterNotebook)
- trip_planner_db - folder that contains MySQL DB and table data (not required to recreate project)
