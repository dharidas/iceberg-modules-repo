# Hands-on With Spark Instructions

Requirements for setting up this environment is having [docker installed](https://www.docker.com).

## Setting Up Environment with Docker Compose

### What is Docker Compose
Docker Compose is a tool that let's us define our environment in a file called `docker-compose.yml`, where we can define multiple containers to run as different 'services' that will get generate on a unique network. 

### Docker Network Trouble Shooting Tips
A dns (domain name service) will be run between the container on the network allowing each container to be referenced by their `container_name` like `http://container_name:8080`. If for some reason the DNS isn't resolving (you errors connecting from one service to another using their container name) then you'll want to use the containers direct ip address on the network docker creates for your environment.

- Find the network by running `docker network ls`

- Then inspect the network with `docker network inspect <network_name>`

This will bring up output that looks like this

```json
[
    {
        "Name": "myproject_default",
        "Id": "6b5d6789abcd",
        "Created": "2024-07-02T12:34:56.789Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Containers": {
            "container_id_1": {
                "Name": "myproject_container_1",
                "EndpointID": "ep1",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            },
            "container_id_2": {
                "Name": "myproject_container_2",
                "EndpointID": "ep2",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]

```

What you want to look at is this section for the ipv4 address of the particular container:

```json
"Containers": {
            "container_id_1": {
                "Name": "myproject_container_1",
                "EndpointID": "ep1",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            },
            "container_id_2": {
                "Name": "myproject_container_2",
                "EndpointID": "ep2",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            }
```

In this case the IP address of the container called `container_id_2` is `172.18.0.3`.

### The Docker Compose File

So in an empty directory somewhere on your computer you will create a `docker-compose.yml` and fill it with the following:

```yaml
version: '3.8'

services:
## Apache Spark + Notebook
  spark:
    platform: linux/x86_64
    image: alexmerced/spark35notebook:latest
    ports: 
      - 8080:8080  # Master Web UI
      - 7077:7077  # Master Port
      - 8888:8888  # Notebook
    environment:
      - AWS_REGION=us-east-1
      - AWS_ACCESS_KEY_ID=admin #minio username
      - AWS_SECRET_ACCESS_KEY=password #minio password
    container_name: spark
    networks:
        spark-iceberg:
## Minio Object Storage
  minio:
    image: minio/minio
    container_name: minio
    ports:
      - "9000:9000"
      - "9001:9001"
    environment:
      - MINIO_ROOT_USER=admin
      - MINIO_ROOT_PASSWORD=password
      - MINIO_DOMAIN=minio
      - MINIO_REGION_NAME=us-east-1
      - MINIO_REGION=us-east-1
    command: server /data --console-address ":9001"
    networks:
        spark-iceberg:
## Nessie Iceberg Catalog
  nessie:
    image: projectnessie/nessie
    container_name: nessie
    ports:
      - "19120:19120"
    networks:
        spark-iceberg:

networks:
  spark-iceberg:
```

Then open a terminal to the same directory and you can use the followin commands to work with this file.

- `docker compose up` run every service
- `docker compose down` shuts down all services
- `docker compose up <service_name>` run a particular serivce
- `docker compose down <service_name>` shuts down a particular service
- `docker compose exec <command> <service_name>` runs specified command within a particular service

_In the terminal output you'll find the url to access the notebook server, that will be needed to get access to the notebook so keep an eye out for it_

### additional breakdown of the docker compose command

# Docker Compose Command Flags

#### Common Flags

- `-f, --file FILE`
  - Specify an alternate compose file (default: `docker-compose.yml`).
  - Example: `docker compose -f custom-compose.yml up`

- `-p, --project-name NAME`
  - Specify an alternate project name (default: directory name).
  - Example: `docker compose -p myproject up`

- `--env-file FILE`
  - Specify an alternate environment file.
  - Example: `docker compose --env-file custom.env up`

#### Service Management

- `up`
  - Build, (re)create, start, and attach to containers for a service.
  - Example: `docker compose up`

- `down`
  - Stop and remove containers, networks, images, and volumes.
  - Example: `docker compose down`

- `start`
  - Start existing containers for a service.
  - Example: `docker compose start`

- `stop`
  - Stop running containers without removing them.
  - Example: `docker compose stop`

- `restart`
  - Restart running or stopped containers.
  - Example: `docker compose restart`

- `build`
  - Build or rebuild services.
  - Example: `docker compose build`

#### Logging and Output

- `logs`
  - View output from containers.
  - Example: `docker compose logs`

- `--no-color`
  - Produce monochrome output.
  - Example: `docker compose --no-color up`

#### Scaling and Resources

- `--scale SERVICE=NUM`
  - Scale a service to a specified number of replicas.
  - Example: `docker compose up --scale web=3`

- `--detach, -d`
  - Run containers in the background.
  - Example: `docker compose up -d`

#### Environment and Dependencies

- `--env-file FILE`
  - Specify an alternate environment file.
  - Example: `docker compose --env-file custom.env up`

- `--no-deps`
  - Don’t start linked services.
  - Example: `docker compose up --no-deps SERVICE`

#### Cleanup and Pruning

- `rm`
  - Remove stopped service containers.
  - Example: `docker compose rm`

- `prune`
  - Remove all stopped containers and networks.
  - Example: `docker compose prune`

#### Debugging and Development

- `--build`
  - Rebuild services even if the image already exists.
  - Example: `docker compose up --build`

- `--force-recreate`
  - Recreate containers even if their configuration and image haven't changed.
  - Example: `docker compose up --force-recreate`

- `--no-build`
  - Don’t build an image, even if it’s missing.
  - Example: `docker compose up --no-build`

- `--abort-on-container-exit`
  - Stops all containers if any container was stopped. Useful in conjunction with `--exit-code-from`.
  - Example: `docker compose up --abort-on-container-exit`

- `--exit-code-from SERVICE`
  - Return the exit code of the selected service container.
  - Example: `docker compose up --exit-code-from web`

#### Help and Version

- `--help`
  - Get help on a command.
  - Example: `docker compose up --help`

- `--version`
  - Show the Docker Compose version information.
  - Example: `docker compose --version`

### Setting Up Minio

Once the minio container is running (`docker compose -d up minio`) head over to `http://localhost:9001` and login using the username `admin` and password `password`. Click on "create a bucket" and create two buckets called "warehouse" and "lakehouse".

### Setting up Nessie

Just start the container `docker compose -d up nessie` and your good to go.

[Project Nessie Documentation](https://www.projectnessie.org)

### Setting Up the Spark Notebook Server

Run the command `docker compose up spark` then look at the terminal output for url like this:

```
http://127.0.0.1:8888/lab?token=efe4625e5f40b8249639b4b0edf90e39ba65b6025ab367e0
```

Put that url in the browser to open up the notebook server. Create a new notebook and add the following code:

```py
import pyspark
from pyspark.sql import SparkSession
import os

## DEFINE SENSITIVE VARIABLES
NESSIE_SERVER_URI = "http://nessie:19120/api/v2"
WAREHOUSE_BUCKET = "s3://warehouse"
MINIO_URI = "http://172.19.0.2:9000"


## Configurations for Spark Session
conf = (
    pyspark.SparkConf()
        .setAppName('app_name')
  		#packages
        .set('spark.jars.packages', 'org.apache.iceberg:iceberg-spark-runtime-3.5_2.12:1.5.2,org.projectnessie.nessie-integrations:nessie-spark-extensions-3.5_2.12:0.91.3,software.amazon.awssdk:bundle:2.20.131,software.amazon.awssdk:url-connection-client:2.20.131')
  		#SQL Extensions
        .set('spark.sql.extensions', 'org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions,org.projectnessie.spark.extensions.NessieSparkSessionExtensions')
  		#Configuring Catalog
        .set('spark.sql.catalog.nessie', 'org.apache.iceberg.spark.SparkCatalog')
        .set('spark.sql.catalog.nessie.uri', NESSIE_SERVER_URI)
        .set('spark.sql.catalog.nessie.ref', 'main')
        .set('spark.sql.catalog.nessie.authentication.type', 'NONE')
        .set('spark.sql.catalog.nessie.catalog-impl', 'org.apache.iceberg.nessie.NessieCatalog')
        .set("spark.sql.catalog.nessie.s3.endpoint",MINIO_URI)
        .set('spark.sql.catalog.nessie.warehouse', WAREHOUSE_BUCKET)
        .set('spark.sql.catalog.nessie.io-impl', 'org.apache.iceberg.aws.s3.S3FileIO')
)

## Start Spark Session
spark = SparkSession.builder.config(conf=conf).getOrCreate()
print("Spark Running")

## TEST QUERY TO CHECK IT WORKING
### Create TABLE
spark.sql("CREATE TABLE nessie.example (name STRING) USING iceberg;").show()
### INSERT INTO TABLE
spark.sql("INSERT INTO nessie.example VALUES ('Alex Merced');").show()
### Query Table
spark.sql("SELECT * FROM nessie.example;").show()
```

this line
```
MINIO_URI = "http://172.19.0.2:9000"
```

Needs to have the correct ip address for your minio container. To get that do the following.

- Find the network by running `docker network ls`

- Then inspect the network with `docker network inspect <network_name>`

From there you will find the details for the minio container to get its ipv4 address.

Now you should be able to run the code and if it runs successfully we are good to go!

### Creating Apache Iceberg Tables

#### Using a Dataframe

```py

from pyspark.sql.functions import current_timestamp

# Create DataFrame
data = [(1, "Alice"), (2, "Bob"), (3, "Catherine")]
columns = ["id", "name"]
df = spark.createDataFrame(data, columns).withColumn("ts", current_timestamp())

# Write DataFrame to Iceberg table
df.writeTo("nessie.df_table").create()
print("Iceberg table created using DataFrames")

#query table
spark.sql("SELECT * FROM nessie.df_table;").show()

```

#### Using SQL

```py
# Create an Iceberg table using SQL
spark.sql("""
    CREATE TABLE nessie.sql_table (
        id INT,
        name STRING
    )
    USING iceberg
""")

spark.sql("INSERT INTO nessie.sql_table VALUES (1, 'Alex Merced'), (2, 'Andew Madson');")

spark.sql("SELECT * FROM nessie.sql_table;").show()
```

### Inserting Data Into an Iceberg Table

#### Dataframe Approach

```py
import json
from pyspark.sql.functions import col

# Step 1: Create a JSON file with a few records
json_data = [
    {"id": 1, "name": "Alice", "ts": "2024-07-02T12:00:00"},
    {"id": 2, "name": "Bob", "ts": "2024-07-02T12:05:00"},
    {"id": 3, "name": "Catherine", "ts": "2024-07-02T12:10:00"}
]

json_file_path = "/tmp/data.json"
with open(json_file_path, 'w') as json_file:
    json.dump(json_data, json_file)

# Step 2: Read the JSON file into a DataFrame
df = spark.read.json(json_file_path)
df = df.withColumn("ts", col("ts").cast("timestamp"))
df.show()

# Step 3: Create the "db" namespace in Nessie
spark.sql("CREATE NAMESPACE IF NOT EXISTS nessie.db")
print("Namespace 'db' created in Nessie")

# Step 4: Create an empty Iceberg table using SQL
spark.sql("""
    CREATE TABLE IF NOT EXISTS nessie.db.example_table (
        id INT,
        name STRING,
        ts TIMESTAMP
    )
    USING iceberg
""")
print("Iceberg table created with no records")

# Step 5: Insert the data from JSON DataFrame into the Iceberg table
df.writeTo("nessie.db.example_table").append()
print("Data inserted into Iceberg table from JSON DataFrame")

spark.sql("SELECT * FROM nessie.db.example_table").show()
```

#### Inserting Using SQL

```py
# Step 1: Create a JSON file with a few records
json_data = [
    {"id": 1, "name": "Alice", "ts": "2024-07-02T12:00:00"},
    {"id": 2, "name": "Bob", "ts": "2024-07-02T12:05:00"},
    {"id": 3, "name": "Catherine", "ts": "2024-07-02T12:10:00"}
]

json_file_path = "/tmp/data.json"
with open(json_file_path, 'w') as json_file:
    json.dump(json_data, json_file)

# Step 2: Read the JSON file into a DataFrame
df = spark.read.json(json_file_path)
df.createOrReplaceTempView("json_table")

# Cast the ts column to timestamp
spark.sql("SELECT id, name, CAST(ts AS TIMESTAMP) AS ts FROM json_table").createOrReplaceTempView("json_table_casted")

# Step 3: Create the "db" namespace in Nessie
spark.sql("CREATE NAMESPACE IF NOT EXISTS nessie.db")
print("Namespace 'db' created in Nessie")

# Step 4: Create an empty Iceberg table using SQL
spark.sql("""
    CREATE TABLE nessie.db.example_sql_table (
        id INT,
        name STRING,
        ts TIMESTAMP
    )
    USING iceberg
""")
print("Iceberg table created with no records")

# Step 5: Insert the data from the temporary view into the Iceberg table
spark.sql("""
    INSERT INTO nessie.db.example_sql_table
    SELECT id, name, ts FROM json_table_casted
""")
print("Data inserted into Iceberg table from JSON DataFrame")

spark.sql("SELECT * FROM nessie.db.example_sql_table").show()
```

#### Using Merge Into

```py
# Step 1: Create a JSON file with a few records
json_data = [
    {"id": 1, "name": "Alice", "count": None, "ts": "2024-07-02T12:00:00"},
    {"id": 2, "name": "Bob", "count": 5, "ts": "2024-07-02T12:05:00"},
    {"id": 3, "name": "Catherine", "count": 10, "ts": "2024-07-02T12:10:00"}
]

json_file_path = "/tmp/data.json"
with open(json_file_path, 'w') as json_file:
    json.dump(json_data, json_file)

# Step 2: Read the JSON file into a DataFrame
df = spark.read.json(json_file_path)
df = df.withColumn("ts", col("ts").cast("timestamp"))
df.createOrReplaceTempView("json_table")
df.show()

# Step 3: Create the "db" namespace in Nessie
spark.sql("CREATE NAMESPACE IF NOT EXISTS nessie.merge_example")
print("Namespace 'merge_example' created in Nessie")

# Step 4: Create an empty Iceberg table using SQL
spark.sql("""
    CREATE TABLE nessie.merge_example.example_table (
        id INT,
        name STRING,
        count INT,
        ts TIMESTAMP
    )
    USING iceberg
""")
print("Iceberg table created with no records")

# Insert the initial data into the Iceberg table
spark.sql("""
    INSERT INTO nessie.merge_example.example_table
    SELECT id, name, count, ts FROM json_table
""")
print("Initial data inserted into Iceberg table")

spark.sql("SELECT * FROM nessie.merge_example.example_table;").show()

# Step 5: Create a source DataFrame for updates
update_data = [
    {"id": 1, "name": "Alice", "count": 1, "op": "increment"},
    {"id": 2, "name": "Bob", "count": 1, "op": "increment"},
    {"id": 3, "name": "Catherine", "count": None, "op": "delete"},
    {"id": 4, "name": "David", "count": 1, "op": "insert"}
]

update_file_path = "/tmp/update_data.json"
with open(update_file_path, 'w') as update_file:
    json.dump(update_data, update_file)

source_df = spark.read.json(update_file_path)
source_df.createOrReplaceTempView("source_table")
source_df.show()

# Step 6: Use the MERGE INTO command to update the Iceberg table
spark.sql("""
    MERGE INTO nessie.merge_example.example_table t
    USING source_table s
    ON t.id = s.id
    WHEN MATCHED AND s.op = 'delete' THEN DELETE
    WHEN MATCHED AND t.count IS NULL AND s.op = 'increment' THEN UPDATE SET t.count = 0
    WHEN MATCHED AND s.op = 'increment' THEN UPDATE SET t.count = t.count + 1
    WHEN NOT MATCHED THEN INSERT (id, name, count, ts) VALUES (s.id, s.name, s.count, current_timestamp())
""")
print("Data merged into Iceberg table using MERGE INTO")

spark.sql("SELECT * FROM nessie.merge_example.example_table;").show()
```

### Partitioning with Spark

```py
# Step 1: Create the "partitioning_example" namespace in Nessie
spark.sql("CREATE NAMESPACE IF NOT EXISTS nessie.partitioning_example")
print("Namespace 'partitioning_example' created in Nessie")

# Step 2: Create an Iceberg table with initial partitioning on the 'name' column
spark.sql("""
    CREATE TABLE nessie.partitioning_example.example_table (
        id INT,
        name STRING,
        count INT,
        ts TIMESTAMP
    )
    USING iceberg
    PARTITIONED BY (name)
""")
print("Iceberg table created with initial partitioning on 'name'")

# Step 3: Insert initial data into the Iceberg table
spark.sql("""
    INSERT INTO nessie.partitioning_example.example_table VALUES
    (1, 'Alice', NULL, TIMESTAMP('2024-07-02T12:00:00')),
    (2, 'Bob', 5, TIMESTAMP('2024-07-02T12:05:00')),
    (3, 'Catherine', 10, TIMESTAMP('2024-07-02T12:10:00'))
""")
print("Initial data inserted into Iceberg table")

# Step 4: Update the partitioning of the Iceberg table to include the 'ts' column
spark.sql("""
    ALTER TABLE nessie.partitioning_example.example_table
    ADD PARTITION FIELD bucket(4, ts) AS ts_bucket
""")
print("Iceberg table partitioning updated to include 'ts' column")

# Step 5: Insert additional data into the Iceberg table
spark.sql("""
    INSERT INTO nessie.partitioning_example.example_table VALUES
    (4, 'David', 15, TIMESTAMP('2024-07-02T12:15:00')),
    (5, 'Eve', 20, TIMESTAMP('2024-07-02T12:20:00'))
""")
print("Additional data inserted into Iceberg table")

spark.sql("SELECT * FROM nessie.partitioning_example.example_table;").show()
```

## Turn Off Your Environment

When done, you can turn off your environment by running `docker compose down` in the same folder as your docker-compose.yml