# Environment Setup

## Docker Compose

- Open up terminal/bash to the directory where you have your docker-compose.yml file which should contain the below:

```yml
version: "3"

services:
  # Nessie Catalog Server Using In-Memory Store
  nessie:
    image: projectnessie/nessie:latest
    container_name: nessie
    networks:
      icebergdataops:
    ports:
      - 19120:19120
  # Minio Storage Server
  minio:
    image: minio/minio:latest
    container_name: minio
    environment:
      - MINIO_ROOT_USER=admin
      - MINIO_ROOT_PASSWORD=password
      - MINIO_DOMAIN=storage
      - MINIO_REGION_NAME=us-east-1
      - MINIO_REGION=us-east-1
    networks:
      icebergdataops:
    ports:
      - 9001:9001
      - 9000:9000
    command: ["server", "/data", "--console-address", ":9001"]
  # Dremio
  dremio:
    platform: linux/x86_64
    image: dremio/dremio-oss:latest
    ports:
      - 9047:9047
      - 31010:31010
      - 32010:32010
      - 45678:45678
    container_name: dremio
    environment:
      - DREMIO_JAVA_SERVER_EXTRA_OPTS=-Dpaths.dist=file:///opt/dremio/data/dist
    networks:
      icebergdataops:
  # Spark
  spark:
    platform: linux/x86_64
    image: alexmerced/spark35nb:latest
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
      icebergdataops:
  #Superset
  superset:
    image: alexmerced/dremio-superset
    container_name: superset
    networks:
      icebergdataops:
    ports:
      - 8088:8088
networks:
  icebergdataops:
```

## Spin Up Environment

*Terminal must be in the same directory as the docker-compose.yml file*

To turn on environment run:

```bash
docker-compose up -d
```

to turn off the environment run:

```bash
docker-compose down
```

to also remove any volumes instead run:

```bash
docker-compose down -v
```

Give it a few minutes for the containers to spin up. You should now be able to access the following in the browser:

- [Dremio](http://localhost:9047)
- [Jupyter Notebook with PySpark and other Data Libraries](http://localhost:8888)
- [Minio](http://localhost:9001)
- [Superset](http://localhost:8088)
*Before using superset you need to initialize by running the command `docker exec -it superset superset init`*

## Creating Buckets in Minio

Head over to [Minio](http://localhost:9001) and create two buckets called `datalake` and `datalakehouse`. The minio username and password are `admin` and `password` respectively.

## Establishing Dremio Connections

Visit [Dremio](http://localhost:9047) and create your dremio username and password, make sure to remember them for use with dbt.

We will add two connections, one to the minio storage server and one to the nessie catalog server.

- The **minio connection** represents our **data lake** where we work with our data as files.

- The **nessie connection** represents our **data lakehouse** where we work with data on our data lake as tables instead of files.

### Minio Connection

- Add an S3 Source:
    - Click on the “Add Source” button again and select S3 from the list of sources.
- Configure the S3 Source for Minio:
    - General Settings:
        - Name: Set the source name to seed.
        - Credentials: Select AWS access key.
        - Access Key: Set to admin (Minio username).
        - Secret Key: Set to password (Minio password).
        - Encrypt Connection: Uncheck this option (since Minio is running locally).
    - Advanced Options:
        - Enable Compatibility Mode: Set to true (to ensure compatibility with Minio).
        - Root Path: Set to `/datalake` (this is where the seed data files are located in Minio).
        - Connection Properties:
            - Set fs.s3a.path.style.access to true.
            - Set fs.s3a.endpoint to minio:9000.
- Save the Source: After entering the configuration details, click Save. The seed bucket is now accessible in Dremio, and you can query the raw data stored in this bucket.

### Nessie Connection

Click "add source" and select "Nessie"

- General settings tab
    - Source Name: nessie
    - Nessie Endpoint URL: http://nessie:19120/api/v2
    - Auth Type: None
- Storage settings tab
    - AWS Root Path: datalakehouse
    - AWS Access Key: admin
    - AWS Secret Key: password
    - Uncheck “Encrypt Connection” Box (since we aren’t using SSL)
    - Connection Properties
        - Key: fs.s3a.path.style.access | Value: true
        - Key: fs.s3a.endpoint | Value: minio:9000
        - Key: dremio.s3.compat | Value: true

## Running Queries