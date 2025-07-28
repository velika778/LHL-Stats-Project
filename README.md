# Final-Project-Statistical-Modelling-with-Python

## Project/Goals

I wanted to see how much I could automate the entire process from start to finish, but ran out of time on cleaning up the final data to build the model so it ended up being fairly manual.

Note that I'm skipping the Yelp portion of the exercise as I don't have a business email to sign up with.

## Process
### City Bikes Data
1. I did an initial API call to determine what cities were available.

2. I decided to focus on Berlin since I know it's very bike-friendly. There are actually 2 bike networks in Berlin so I chose to focus only on one, Nextbike.

3. I did an API call for Nextbike and exported the normalized data to csv.

4. I put the data into a dataframe and cleaned it up:
	
	i. Replaced NaN in the bike/slots columns with 0 since it implies there are no bikes/slots available.

	ii. Dropped timestamp since it refers to the time that I pulled the data, and would only be meaningful here if I had pulled it at different times for comparison. By only pulling it once, we are assuming that timestamp is irrelevant to our conclusions.

	iii. Added a column for total bikes, which is a sum of free bikes and empty slots.

5. I exported the final results to csv to work with later.

### FourSquare Data

1. I imported the City Bikes data into a data frame that included only the name of the station and its latitude and longitude.

2. I used the latitude and longitude of each station to create a looped API call using each pair of coordinates as the variable.

3. Each API loop would extract Places data within 1000m of the given coordinates and produce a json response. I saved each response to its own json file so I wouldn't have to call it again.

** I probably could have appended each response to a single json file but I couldn't figure it out in time!

4. Once I had the data for each coordinate saved, I merged all the json files into a single file for ease of extraction.

5. My biggest struggle was that the POI data came in as a mix of lists and dictionaries, so I initially found it difficult to iterate over them.

6. I put all the data into a dataframe and exported to csv to work with later.

### Joining Data

1. I imported the City Bikes and FourSquare data into separate dataframes.

2. I created a function that would round latitude and longitude to a given decimal point and concatenate them into a new column called "latlon_bucket". I ran this function on both dataframes.

	i. I initially tried it with the coordinates rounded to 3 decimal places but got too many missing values when I moved on to the next step, so I ultimately rounded to only 2 decimal places.

3. I then did a left join on the 2 dataframes using the latlon_bucket column.

4. I checked the new dataframe for missing values. I dropped postcode, name and address where they were missing since this implies they were invalid locations.

5. I also dropped the unique City Bikes and FourSquare keys since they aren't needed anymore.

6. I renamed the columns in the new dataframe to make it easier for me to read, and saved it to csv.

### EDA on joined data

1. I created a histogram to confirm that no bike stations were further than 1000m or less than 0m from the list of venues.

![Berlin bike station distance histogram](/images/berlin_hist.png)

2. I created 2 boxplots to confirm that no venues or bike stations were outside of the expected latitude/longitude distribution for Berlin.

![Berlin latitude](/images/berlin_latitude.png)

![Berlin longitude](/images/berlin_longitude.png)

### Creating the SQL database

1. I created 2 tables in sqlite3, 1 for the CityBikes data and 1 for FourSquare, and loaded them from the dataframes

2. I confirmed that the shape of each table matched the shape of each original dataframe to confirm nothing had gone wrong while loading them.

3. I joined the 2 tables in SQL using latlon_bucket as I had done in Python to confirm it produced the same shape as the joined dataframe I had been working with. It did so except for 1 extra column since SQL duplicates the joined column and Python does not.

### Building the model

1. I imported my joined dataframe from the "Joining Data" step.

2. Since there is a ton of granularity in the category names provided by FourSquare, I created a helper column that groups each venue into a meta category, called "category group".

3. I then pivoted on the data, counting the number of venues within each category group per bike station and the number of bikes available. 

4. I then created a linear regression model on this pivoted data using the bike station as a constant, and the number of bikes as my dependent variable.

## Results

Based on my results, one might assume there's a correlation between the number of nearby restaurants and the number of bikes available at a given station in Berlin. This seems improbable but I have no way of verifying that based on the data available.

## Challenges 

JSON files with inconsistent types of nesting!

## Future Goals

I would like to figure out how to create a dataframe similar to the pivot table I ended up creating, without having to pivot on it to begin with. I wanted to create a dataframe that had each value from the category name column as its own column, then count the number of venues within that category per bike station, without having to feed the venue name into the dataframe to begin with.