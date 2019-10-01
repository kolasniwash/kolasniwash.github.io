---
title: "Creating datasets from bigdata with google big query"
date: 2019-10-01T11:39:30-04:00
---

Summary
This post is a summary of what I've learned from the Launching into Machine Learning course by Google Cloud on Coursera. It's part of the Machine Learning with TensorFlow on Google Cloud Platform Specialization. Code examples are taken from the [training-data-analyist](https://github.com/nicholasjhana/training-data-analyst) repo maintained by google cloud.

#Intro
When starting into machine learning one of the first things I learned was how to split my data into training and test sets. And how important it is this is done in a repeatable way. At first this seemed strightforward. I could use functions like train_test_split from sklearn and like magic I have train and test datasets.

Moving further in my machine learning journy I learned it wasn't always that simple. I realized many of the datasets I was working with were small, usually on the order of 50k samples. What If the dataset was much larger? Like 10x or 100x larger? How would I work with my data when it won't fit into my computers memory.

From previous work with SQl I knew there were reasonable solutions to extract subsets of my data from a larger source. I wanted a clean and structured way to do this.

#Classic way to set up training and test datasets
Using train_test_split is the ubiqutous method machine learning practitioners use to segment a dataset. For datasets sized less than about 100k rows most computers can hold the whole dataset in their memory and run something like the following.  

```python
from sklearn.model_selection import train_test_split
X_train, X_test, y_train, y_test = train_test_split(x, y, test_size = 0.2, random_state = 0)
```
_SciKit learn method to create train and test datasets_

It's a clean and simple way to get an 80/20 split on the dataset x with target dataset y. And when we set the random_state field we always produce the same subsets of data in each of the train and test buckets.

A variation on this method is to split the dataset into train, validation, and test. This technique is seen in deep learning where cross validation methods are computaionally intentse and costly. To do this we first split the training and test then the training into training and validation.

```python
from sklearn.model_selection import train_test_split

#split the dataset into training and test
X_train, X_test, y_train, y_test = train_test_split(x, y, test_size = 0.2, random_state = 0)

#split the training dataset into training and validation
X_train, X_val, y_train, y_val = train_test_split(X_train, y_train, test_size=0.25, random_state=0)
```
_SciKit learn method to create train, validate, and test datasets_

This is a simple quick solution if you have enough data to spare. I.e. A 100k row dataset would leave you with datasets of 60k training, 20k validation, and 20k test. 

However it's probably not a good solution if your data absolutely massive like the 92M rows in the [Chicago Taxi Trips](https://www.kaggle.com/chicago/chicago-taxi-trips-bq) dataset on kaggle. Working with this size of data the above method won't work. 

#Method for selecting a data subset
The general approcah to working with big data sets is to first develop a model on a smaller subset of the data, like the 100k size explained above. How can we select this data from the larger dataset?

First, we will assume the dataset is hosted in structured database. This could be a private company database or more likely a cloud hosted service like Google Big Query (GBQ) or Amazon Redshift. We can use SQL to query the data and select a subset. But what's a reasonable way to do this? Lets look at some options.

##Query the data as a single block
We might consider selecting 100k of our larger dataset based on pickup_datetime, selecting only data within certain dates, or by location, only selecting a certain area. This is a valid approach if the subset you select is representative of the actual data you plan to use to make predictions. I.e. if you only plan to make predictions on taxi rides in Brooklyn you might be able to sample rides that begin and end in Brooklyn and arrive at a small enough sample. 

```SQL
SELECT *
FROM `bigquery-public-data.new_york_taxi_trips.tlc_green_trips_2015`
WHERE pickup_latitude >= 40.768297 
      AND pickup_latitude <= 40.806139
      AND pickup_longitude <= -73.958011 
      AND pickup_longitude >=-73.994169
      AND dropoff_latitude >= 40.768297 
      AND dropoff_latitude <= 40.806139
      AND dropoff_longitude <= -73.958011 
      AND dropoff_longitude >=-73.994169 
```
_Query that returns taxi rides that start and terminate in New York's Upper West Side. Approx 88k rows._

For any other application however, this method introduces obvious bias into the data. Our sample would likely not have the same representative distribution as the original. It would only be useful for highly specific cases. How can we run a model that makes predictions for fare_amount if we train on one neighbourhood and test on another?

## Query data with a random function
Like in the SciKit Learn example above we could use a RAND() call in the SQL query to select rows as a random sample. This would allow us to generate a sample dataset with the same distribution as the original.

```SQL
SELECT COUNT(*)
FROM (
  SELECT RAND() as split_feature,
         pickup_datetime,
         dropoff_datetime
  FROM `bigquery-public-data.new_york_taxi_trips.tlc_green_trips_2015`
)
WHERE split_feature < 0.01
```

_Adapted from the [google cloud training-data-analist repo](https://github.com/nicholasjhana/training-data-analyst/blob/master/courses/machine_learning/deepdive/02_generalization/repeatable_splitting.ipynb)_

In the above query we generate a random number with RAND() in the split_feature column. We then only select 1% of all rows with the WHERE split_features < 0.01 which returns 192,389 rows. This is a resonable dataset to work with for model development. From here we could download the dataset as a csv and work with it on our machine, using sklearn to do the train and test spliting for us.

The downside to this model however is that we could not run this query again and regenerate the same data. Why not? Because the RAND() function will always generate different real number values per row. Unlike the sklearn implementation, SQL does not have a random_state feature that lets us fix the random intervals being genreated. 

So this method is not robust enough for a end-to-end production pipeline.

## Query using a MOD operator 
A more robust solution, and one I learned from the [Launching into Machine Learning](https://www.coursera.org/learn/launching-machine-learning?) Coursera course, is to transform a column in the dataset with a hash function and then filter it with a MOD operator. This method has the benefit of always being repeatable and therefore selecting the same data when the query is run multiple times.

Lets take a look at what this looks like in SQL.

```SQL
SELECT MOD(ABS(FARM_FINGERPRINT(CAST(pickup_datetime as string))), 1500) as hash_value, 
       *
FROM `bigquery-public-data.new_york_taxi_trips.tlc_green_trips_2015`
WHERE MOD(ABS(FARM_FINGERPRINT(CAST(pickup_datetime as string))), 1500) < 8
```
_Query using a hash to select 'random' row values. Returns approx 100k rows. Adapted from the [google cloud training-data-analist repo](https://github.com/nicholasjhana/training-data-analyst/blob/master/courses/machine_learning/deepdive/02_generalization/repeatable_splitting.ipynb)_




### How is this query working?

1. The query uses the [```FARM_FINGERPRINT()```](https://cloud.google.com/bigquery/docs/reference/standard-sql/hash_functions) function to transform a string, in this case the datetime of pickup, into a hash value.
2. On the hash value we apply he MOD operator returning the remainder by the value by which we wish to reduce the dataset, in this case 1500.
3. In the WHERE clause we select reminders from 0 to 7 as our sample. We could choose any values

We can calculate how many rows we expect to be returned by different values in the MOD function: (19233765 rows / 1500) * 8 segments ~ 102k

### Benefits of this method
- 


vendor_id
pickup_datetime
dropoff_datetime
store_and_fwd_flag
rate_code
pickup_longitude
pickup_latitude
	dropoff_longitude
	dropoff_latitude
	passenger_count
	trip_distance
	fare_amount
	extra
	mta_tax
	tip_amount
	tolls_amount
	ehail_fee
	total_amount
	payment_type
	distance_between_service
	time_between_service
	trip_type
  imp_surcharge	


[NYC TLC Trips public dataset from google](https://console.cloud.google.com/marketplace/details/city-of-new-york/nyc-tlc-trips?filter=solution-type:dataset&q=NY&id=e4902dee-0577-42a0-ac7c-436c04ea50b6)








1. How I see most train test splits on-line in turotials
  - small data set, easily managed on one computer
  - example of a train/test split 
  - if you have a small data set probably this method is overkill
  - if working with a live dataset, ie big data in the neighbourhood of 1M rows or more this is likely helpful
2. What you'll learn
  - How to connect to google big query from a cloud hosted jupyter notebook
  - how to write a SQL query that returns a training, validation, test split
3. Quick Recap on Training, validation, Test datasets
4. How we can use GBQ to split data
  - Method descrption: 
    - using rand doesn't work becuase no random-key.
    - use a column in the data as a unique hash
    - call mod on the hash and set a threshol value
5. Pros and cons of this method
  - Pros:
    - able to work with huge amounts of data and reduce them into a manageable size for building/evaluating inital models
    - is repeatable, so other data scientists can run your code and be sure they are loading the same data
  - Cons:
    - overkill for a small dataset
    - while the same segments of data are always present if the underlying data is updated the train/validate/test sets will have changed




This project investigates various model implementations that predict short-term (24 hour advance) energy demands in the Spanish energy market.

[Watch the presentation](https://youtu.be/KaWCwBD_UBA) 


## Problem Definition and Motivation
This project is inspired by the paper [Tackling Climate Change with Machine Learning](https://arxiv.org/abs/1906.05433) where forecasting is identified as one of the highest impact research areas to contributing to more renewable energy in the grid. Further, it explores the results from [here](https://www.researchgate.net/publication/330155110_Short-Term_Load_Forecasting_in_Smart_Grids_An_Intelligent_Modular_Approach) where the authors argue that traditional statistical forecasting is more computationally efficient compared to state of the art approaches. Finally, the last model draws from the data structure, and problem setup in the [following paper](https://www.researchgate.net/publication/323847484_Statistical_and_Machine_Learning_forecasting_methods_Concerns_and_ways_forward) to implement a state of the art Long Short Term Network forecast.
