# dbt setup

## Create a Python Virtual Env

From terminal:

- `python3 -m venv venv`

- `source venv/bin/activate`

## Install dbt

- `pip install dbt-dremio`

## Create a dbt project

- `dbt init your_project_name`

- Configure your dbt project

    - **Enter a Number:** Enter the number that corresponds to Dremio which should be “1” (this tells dbt it is using the Dremio plugin).
    - **Enter a Number:** The next number is which version of Dremio you are using, which for this tutorial should be Dremio Cloud which is number “1” (choose Dremio Software with Username/Password if working from laptop dremio deployment).
    - **Choose an API:** Identify which Dremio Cloud API are you using; default uses the North American API, select the European one if using the European Dremio Cloud. (If working from your laptop use 127.0.0.1 as the host and 9047 as the port.)
    - **Enter Email:** This is the email your individual Dremio account is under. (If working from a laptop enter username, then password, then skip to object_storage_source.)
    - **Enter PAT Token:** Generate a PAT token from the Dremio Cloud UI and enter it here for this step.
    - **Enter Project ID:** Enter the ID of the Dremio Cloud project this dbt project will run against.
    - **object_storage_source:** This should be the name of an Arctic catalog or object storage source in your project (e.g., S3). (On your laptop, this could be any Nessie/metastore/object storage source.)
    - **object_storage_path:** This would be the path to a sub-location in your object storage for materializing tables (new physical tables are saved here). (Example: tests.jan92024)
    - **dremio_space:** For Dremio Cloud, this would be the name of the Arctic catalog you want views to be added to (example: Arctic). (If working from a laptop, this would be the name of a space you created on your account.)
    - **dremio_space_path:** A subfolder of the Arctic catalog you want views created by your dbt models to appear in (example: dbt_practice).


## Configuring dbt


## writing dbt models


## running dbt