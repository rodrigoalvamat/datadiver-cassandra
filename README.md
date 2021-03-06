# Apache Cassandra Database Modeling & ETL Pipeline
<br/>

>Author: Rodrigo de Alvarenga Mattos
>
>June 21, 2022
>
> [Data Engineering Nanodegree - Udacity](https://www.udacity.com/course/data-engineer-nanodegree--nd027)

<br/>

## Introduction

The goal of this project is to create a NoSQL database solution optimized for queries and analysis of users' song play activities for the Sparkify streaming service. 

This new design solved the difficulty of performing analytical tasks with information from the CSV log and metadata files.

<br/>

## Project Dependencies

- [Python 3.10](https://www.python.org) 
- [Cassandra Driver 3.25.0](https://pypi.org/project/cassandra-driver) 
- [Jupyter Lab 3.4.3](https://pypi.org/project/jupyterlab)
- [Ipykernel 6.15.0](https://pypi.org/project/ipykernel)
- [Pandas 1.4.2](https://pandas.pydata.org)
- [Pdoc3 12.0.2](https://pdoc3.github.io/pdoc)

<br/>

## Docker Image

Start the project container using the docker run command:

```console
docker run --name datadiver-cassandra --hostname localhost -p 9042:9042 -p 80:80 -d datadiverdev/cassandra-jupyter
```

| Action               | Link                                           | Description                               |
|----------------------|------------------------------------------------|-------------------------------------------|
| **Open Jupyter Lab** | [http://localhost/lab](http://localhost/lab)   | Run python scripts and jupyter notebooks. |    

<br/>

## CSV Data Files Schema

<br/>

1. **Event Dataset** - The CSV files, located in the directory [data/event_data](./data/event_data), are a subset of the [Million Song Dataset](http://millionsongdataset.com) with data generated by an [event simulator](https://github.com/Interana/eventsim).
2. Each file contains the following data schema:
   

| artist | auth | firstName | gender | itemInSession | lastName | length | level | location | method | page | registration | sessionId | song | status | ts | userId |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
Des'ree | Logged In | Kaylee | F | 1 |Summers | 246.30812 | free | "Phoenix-Mesa-Scottsdale, AZ" | PUT | NextSong | 1.54034E+12 | 139 | You Gotta Be | 200 | 1.54111E+12 | 8 |

<br/>

## Database Schema Design

The database was modeled with three tables to fit each query requirement as defined by the project specification. In addition, an automated column matching process was used to find possible unique key combinations. Take a look at the Jupyter Notebook [etl.ipynb](./etl.ipynb) first section to get detailed information on the design process.

All the CQL types and tables were defined in the [lib/etl/queries.py](./lib/etl/queries.py) file.

The table below shows the tables' structures and the key definitions:

| Table Name        | Table Columns                                                       | Partition, Composite & Clustering Key |
|-------------------|---------------------------------------------------------------------|---------------------------------------|
| songplayBySession | artist, song, length, sessionId, itemInSession                      | ((sessionId, itemInSession))          |
| songplayByUser    | artist, song, firstName, lastName, userId, sessionId, itemInSession | ((userId, sessionId), itemInSession)  |
| songplayBySong    | song, sessionId, itemInSession, firstName, lastName                 | (song, sessionId, itemInSession)      |

**See Also:** [Cassandra Partition Key, Composite Key, and Clustering Key](https://www.baeldung.com/cassandra-keys)

<br/>

## Creating the Database

The pseudo-code below shows the main pipeline of the database creation process in the [lib/etl/pipeline.py](./lib/etl/pipeline.py) script:

```python
database.create_session('hostname')
database.create_keyspace('keyspace')
database.execute_queries(drop_table_queries)
database.execute_queries(create_table_queries)
```
<br/>

**Note:** You can set the *--hostname* and *--keyspace* command line arguments when running the [etl.py](./etl.py) script.

## ETL Pipeline Development

The ETL processes were developed in **two phases**. The first one implements the **extraction and transformation** of the CSV data files. The second one is the **batch loading processing** of CQL insert statements according to the [DataStax documentation](https://docs.datastax.com/en/cql-oss/3.3/cql/cql_using/useBatchGoodExample.html).

The table below shows the sequence of the ETL pipeline processes:

| Process                  | Operation                            | Result                                   | Code File                         |
|--------------------------|--------------------------------------|------------------------------------------|-----------------------------------|
| Process CSV source files | Loop through each file               | In-memory list of data rows              | [csv_data](./lib/etl/csv_data.py) |
| Create CSV target file   | Loop through each data row & filter  | Single CSV file without null artists     | [csv_data](./lib/etl/csv_data.py) |
| Map CSV to CQL values    | Replace single quote char to unicode | Sanitized CQL values                     | [pipeline](./lib/etl/pipeline.py) |
| Map CSV to CQL values    | Table column selection               | List of CQL columns to insert            | [pipeline](./lib/etl/pipeline.py) |
| Batch Insert             | Write CQL inserts to the buffer      | Commit when reaching MAX items and reset | [batch](./lib/etl/batch.py)       |
| Shutdown                 | Close database connection            | Print time statistics                    | [pipeline](./lib/etl/pipeline.py) |

<br/>

**Note:** Take a look at the notebook [etl.ipybn](./etl.ipynb) to see the step-by-step development of each ETL process described above.

<br/>

**Run the command** below to execute the ETL pipeline:

```bash
# from the root directory of this project

# run with default settings
python -m etl 

# print the command line --help
python -m etl --help

# set any optional command line arguments
python -m etl --batchsize=SIZE --sourcedir=DIR --targetfile=FILE --hostname=HOST --keyspace=KEY
```

<br/>

## Auto-generate API Documentation

The [pdoc](https://pdoc3.github.io/pdoc/) documentation generator was used to output the [HTML docs](https://htmlpreview.github.io/?https://github.com/rodrigoalvamat/datadiver-cassandra/blob/main/docs/lib/index.html) from the source code ```DOCSTRIGS```.
