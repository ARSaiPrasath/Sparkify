# CONTEXT

A startup called Sparkify wants to analyze the data they've been collecting on songs and user activity on
their new music streaming app. The analytics team is particularly interested in understanding what
songs users are listening to. Currently, they don't have an easy way to query their data, which resides in
a directory of JSON logs on user activity on the app, as well as a directory with JSON metadata on the
songs in their app.

They'd like a data engineer to create a Postgres database with tables designed to optimize queries on
song play analysis, and bring you on the project. 

Your role is to create a database schema and ETL
pipeline for this analysis. You'll be able to test your database and ETL pipeline by running queries given
to you by the analytics team from Sparkify and compare your results with their expected results.

# DATA
1. Song datasets: All json files are located in the song_data folder. Sample of the file:
{"num_songs": 1, "artist_id": "ARJIE2Y1187B994AB7", "artist_latitude": null, "artist_longitude": null, "artist_location": "", "artist_name": "Line Renaud", "song_id": "SOUPIRU12A6D4FA1E1", "title": "Der Kleine Dompfaff", "duration": 152.92036, "year": 0}
2. Log datasets: All json files are located in the log_data folder. Sample of the file:
{"artist":"Slipknot","auth":"Logged In","firstName":"Aiden","gender":"M","itemInSession":0,"lastName":"Ramirez","length":192.57424,"level":"paid","location":"New York-Newark-Jersey City, NY-NJ-PA","method":"PUT","page":"NextSong","registration":1540283578796.0,"sessionId":19,"song":"Opium Of The People (Album Version)","status":200,"ts":1541639510796,"userAgent":"\"Mozilla\/5.0 (Windows NT 6.1) AppleWebKit\/537.36 (KHTML, like Gecko) Chrome\/36.0.1985.143 Safari\/537.36\"","userId":"20"}

#DATABASE SCHEMA
The schema used for this exercise is the Star Schema: There is one main fact table containing all the measures associated to each event (user song plays), and 4 dimentional tables, each with a primary key that is being referenced from the fact table.
Fact Table
songplays - records in log data associated with song plays i.e. records with page NextSong

songplay_id (INT) PRIMARY KEY: ID of each user song play
start_time (DATE) NOT NULL: Timestamp of beggining of user activity
user_id (INT) NOT NULL: ID of user
level (TEXT): User level {free | paid}
song_id (TEXT) NOT NULL: ID of Song played
artist_id (TEXT) NOT NULL: ID of Artist of the song played
session_id (INT): ID of the user Session
location (TEXT): User location
user_agent (TEXT): Agent used by user to access Sparkify platform
Dimension Tables
users - users in the app

user_id (INT) PRIMARY KEY: ID of user
first_name (TEXT) NOT NULL: Name of user
last_name (TEXT) NOT NULL: Last Name of user
gender (TEXT): Gender of user {M | F}
level (TEXT): User level {free | paid}
songs - songs in music database

song_id (TEXT) PRIMARY KEY: ID of Song
title (TEXT) NOT NULL: Title of Song
artist_id (TEXT) NOT NULL: ID of song Artist
year (INT): Year of song release
duration (FLOAT) NOT NULL: Song duration in milliseconds
artists - artists in music database

artist_id (TEXT) PRIMARY KEY: ID of Artist
name (TEXT) NOT NULL: Name of Artist
location (TEXT): Name of Artist city
lattitude (FLOAT): Lattitude location of artist
longitude (FLOAT): Longitude location of artist
time - timestamps of records in songplays broken down into specific units

start_time (DATE) PRIMARY KEY: Timestamp of row
hour (INT): Hour associated to start_time
day (INT): Day associated to start_time
week (INT): Week of year associated to start_time
month (INT): Month associated to start_time
year (INT): Year associated to start_time
weekday (TEXT): Name of week day associated to start_time

# PROJECT STRUCTURE
Files used on the project:

1. data folder nested at the home of the project includes the song and log data, where all needed jsons reside.
2. sql_queries.py contains all your sql queries, and is imported into the files bellow.
3. create_tables.py drops and creates tables. You run this file to reset your tables before each time you run your ETL scripts.
4. test.ipynb displays the first few rows of each table to let you check your database.
5. etl.ipynb reads and processes a single file from song_data and log_data and loads the data into your tables.
6. etl.py reads and processes files from song_data and log_data and loads them into your tables.
7. README.md current file, provides discussion on my project.

# ETL PIPELINE(etl.py)
## FOR song_data
1. We start our pipeline by connecting it to the sparkify database.
2. Then we read song_data folder where all the json data of the users. While encountering each json file we use the function process_song_file. 
3. We load the file as a dataframe using pandas function "read_json()"
4. For each row in the dataframe we select the fields we are interested in:
 ```
 song_data = [song_id, title, artist_id, year, duration]
 ```
 ```
  artist_data = [artist_id, artist_name, artist_location, artist_longitude, artist_latitude]
 ```
5. We insert this data into the respective databases.

## FOR log_data
1. We load our data as a dataframe same way as with songs data.
2. We select rows where page = 'NextSong' only.
3. We convert ts column where we have our start_time as timestamp in millisencs to datetime format. We obtain the parameters we need from this date (day, hour, week, etc), and insert everything into our time dimentional table.
4. We store user data into our user table
5. Finally we lookup song and artist id from their tables by song name, artist name and song duration that we have on our song play data.
6. The final step is to insert all our data into our songplay fact table.
