## Description

In this project we model user activity data with Postgres and build an ETL
pipeline using Python. We define fact and dimension tables for a star schema,
and write an ETL pipeline that transfers data from files in two local
directories into these tables.


## Usage

Create a postgres database and tables in a star schema:
```bash
python create_tables.py
```

Then start the ETL pipeline:
```bash
python etl.py
```

## Data Sources

### Song Dataset

The first dataset is a subset of real data from the [Million Song
Dataset](https://labrosa.ee.columbia.edu/millionsong/). Each file is in JSON
format and contains metadata about a song and the artist of that song. The
files are partitioned by the first three letters of each song's track ID. For
example, here are filepaths to two files in this dataset.

```bash
song_data/A/B/C/TRABCEI128F424C983.json
song_data/A/A/B/TRAABJL12903CDCF1A.json
```

And below is an example of what a single song file, TRAABJL12903CDCF1A.json,
looks like.

```javascript
{
   "num_songs":1,
   "artist_id":"ARJIE2Y1187B994AB7",
   "artist_latitude":null,
   "artist_longitude":null,
   "artist_location":"",
   "artist_name":"Line Renaud",
   "song_id":"SOUPIRU12A6D4FA1E1",
   "title":"Der Kleine Dompfaff",
   "duration":152.92036
}
```

### Log Dataset

The second dataset consists of log files in JSON format generated by this
[event simulator](https://github.com/Interana/eventsim) based on the songs in
the dataset above. These simulate activity logs from a music streaming app
based on specified configurations.

The log files in the dataset are partitioned by year and month. For example,
here are filepaths to two files in this dataset.

```bash
log_data/2018/11/2018-11-12-events.json
log_data/2018/11/2018-11-13-events.json
```

And below is an example of what the data in a log file, 2018-11-12-events.json,
looks like.

| artist       | auth      | firstName | gender | itemInSession | lastName |   length | level | location                             | method | page     |      registration | sessionId | song                             | status |                ts | userAgent                                                                                                                  | userId |
| ------------ | --------- | --------- | ------ | ------------- | -------- | -------- | ----- | ------------------------------------ | ------ | -------- | ----------------- | --------- | -------------------------------- | ------ | ----------------- | -------------------------------------------------------------------------------------------------------------------------- | ------ |
|              | Logged In | Kevin     | M      |         False | Arellano |          | free  | Harrisburg-Carlisle, PA              | GET    | Home     | 1.540.006.905.796 |       514 |                                  |    200 | 1.542.069.417.796 | "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.125 Safari/537.36" |     66 |
| Fu           | Logged In | Kevin     | M      |          True | Arellano | 280,058… | free  | Harrisburg-Carlisle, PA              | PUT    | NextSong | 1.540.006.905.796 |       514 | Ja I Ty                          |    200 | 1.542.069.637.796 | "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.125 Safari/537.36" |     66 |
|              | Logged In | Maia      | F      |         False | Burke    |          | free  | Houston-The Woodlands-Sugar Land, TX | GET    | Home     | 1.540.676.534.796 |       510 |                                  |    200 | 1.542.071.524.796 | "Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.143 Safari/537.36"            |     51 |
| All Time Low | Logged In | Maia      | F      |          True | Burke    | 177,841… | free  | Houston-The Woodlands-Sugar Land, TX | PUT    | NextSong | 1.540.676.534.796 |       510 | A Party Song (The Walk of Shame) |    200 | 1.542.071.549.796 | "Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/36.0.1985.143 Safari/537.36"            |     51 |


## Schema for Song Play Analysis

This is our star schema optimized for queries on song play analysis. This
includes the following tables.

### Fact Table

    1. songplays - records in log data associated with song plays
        * songplay_id, start_time, user_id, level, song_id, artist_id,
          session_id, location, user_agent

### Dimension Tables

    2. users - users in the app
        * user_id, first_name, last_name, gender, level

    3. songs - songs in music database
        * song_id, title, artist_id, year, duration

    4. artists - artists in music database
        * artist_id, name, location, latitude, longitude

    5. time - timestamps of records in songplays broken down into specific units
        * start_time, hour, day, week, month, year, weekday
