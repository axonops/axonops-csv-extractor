# AxonOps™ CSV Extractor

The AxonOps CSV Extractor is a Python utility designed to extract monthly data from AxonOps and convert it to CSV files. This tool is particularly useful for integrating into data pipelines, data warehouses or ETL workflows

You simply add your AxonOps API keys as environment variables, create a report config (see `data/queryconfig/query_config_example.json`) and then run `axonops_csv_extractor.py`.

To get it working, load the Python dependencies in `requirements.txt` into your Python environment and then run the Python script `axonops_csv_extractor.py`
```
usage: axonops_csv_extractor.py [-h] -o OUTPUTDIR -q QUERYCONFIG -m MONTHOFYEAR [-d]

AxonOps CSV Extractor

options:
  -h, --help            show this help message and exit
  -o OUTPUTDIR, --outputdir OUTPUTDIR
                        The file path to a directory for outputting the CSV data.
  -q QUERYCONFIG, --queryconfig QUERYCONFIG
                        File path to the JSON configuration file listing the queries to run and extract to CSV. See the README.md for more information on this configuration file.
  -m MONTHOFYEAR, --monthofyear MONTHOFYEAR
                        The month of year in format YYYYMM for which data will be extracted to CSV. This can not be in the future nor the current month
  -d, --deletejson      If set, the downloaded JSON will be kept in the output directory. By default it is automatically deleted after being converted to CSV.
```
**Note:** please ensure that there is sufficient disk space at the location you choose to output the CSV - they can be large.


For example
```bash
python axonops_csv_extractor.py --outputdir data/results/mydata --queryconfig data/queryconfig/myqueries.json --monthofyear 202409
```

This will extract all the data for September 2024 returned by the queries in myqueries.json and store it as CSV files in data/results/mydata

***
- [Setup Instructions](#setup-instructions)
  - [Query Configuration Setup](#query-configuration-setup)
    - [JSON Field Information](#json-field-information)
  - [Python3 Setup and AxonOps API Key](#python3-setup-and-axonops-api-key)
    1. [Clone the Repository](#1-clone-the-repository)
    2. [Generate an AxonOps API token for your organisation](#2-generate-an-axonops-api-token-for-your-organisation)
    3. [Set-up your AxonOps organisation and API tokens as environment variables](#3-set-up-your-axonops-organisation-and-api-tokens-as-environment-variables)
    4. [Set Up a Python Virtual Environment](#4-set-up-a-python-virtual-environment)
    5. [Activate the Virtual Environment](#5-activate-the-virtual-environment)
    6. [Install Required Python Packages](#6-install-required-python-packages)
    7. [Run the Python Script](#7-run-the-python-script)
    8. [Deactivate the Virtual Environment](#8-deactivate-the-virtual-environment)


## Setup Instructions

### Query Configuration Setup

The `--queryconfig` argument should be pointing at a JSON file the defines the queries to run, the results of which will be converted to CSV. There is an example you can see in `data/queryconfig/query_config_example.json`.

The file looks roughly like this:

```JSON
{
    "clusters": ["mycassandracluster1","mycassandracluster2","mycassandracluster3"],
    "queries": [
        {
            "description": "Live Disk Space Used by DC and Keyspace",
            "unit": "bytes (SI)",
            "axon_query": "sum(cas_Table_LiveDiskSpaceUsed{function='Count',keyspace!~'system|system_auth|system_distributed|system_schema|system_traces',scope!=''}) by (dc, keyspace)",
            "file_prefix": "live_disk_per_keyspace"
        },
        {
            "description": "Total Coordinator Reads by DC and Keyspace",
            "unit": "rps",
            "axon_query": "sum(cas_Table_CoordinatorReadLatency{axonfunction='rate',function=~'Count',keyspace!~'system|system_auth|system_distributed|system_schema|system_traces'}) by (dc,keyspace,scope)",
            "file_prefix": "total_coordinator_table_reads_per_dc",
            "field_renames": [
                {"rename": "scope", "value": "table"}
            ]
        }
    ]
}
```

#### JSON Field Information

* ***clusters***: Add the names of your clusters connected to AxonOps in the `clusters` list.
* ***queries***: For each of the clusters in entered, this is the list of queries that will be run and the results converted to CSV.
  *  ***description***: A human readable description of the query
  *  ***unit***: the unit returned
  *  ***axon_query***: the AxonOps query to run the results of which will be converted to CSV.
  *  ***file_prefix***: The file prefix that will be used in the name of the CSV. This must be unique.
  *  ***field_renames***: (optional) Sometimes, the JSON API response has fields returned that can be named better. 


### Python3 Setup and AxonOps API Key

You need to install Python 3 for this application, a good resource is the official Python website. You can find installation instructions and download links for various operating systems there. 
- [Python Downloads](https://www.python.org/downloads/) - This page provides the latest releases of Python for Windows, macOS, and other platforms, along with detailed installation instructions.

#### **1. Clone the Repository**

First, clone the GitHub repository to your local machine:

```bash
git clone https://github.com/axonops/axonops-csv-extractor.git
cd axonops-csv-extractor
```

#### **2. Generate an AxonOps API token for your organisation**

Login to your AxonOps console. For the SaaS console go to http://console.axonops.com. 

Once you have logged in, choose the organisation you want to create an API token for. Then enter the API Tokens section:

<img width="1076" alt="API Tokens" src="https://github.com/user-attachments/assets/6533429a-892b-4a46-90d9-93b3617c5660">

From there, click create a new API Token. Give the token a name, expiry period, select the clusters you want to be able to access with the token and select the Readonly Role

<img width="602" alt="Create New Token" src="https://github.com/user-attachments/assets/7cfc852e-5797-464d-afdf-7228d4b39dd4">

Click the Generate button and then you can copy the API token to use in Step 3.

<img width="604" alt="Generated Token" src="https://github.com/user-attachments/assets/4632bd45-e444-46c9-b139-13aca0ec3924">

#### **3. Set-up your AxonOps organisation and API tokens as environment variables**

To interact with the AxonOps APIs to query its data you need to store your organisation and API key as environment variables in a new `.env` you need to create in the root directory of the repo. 

See `.env-example` - copy this to a file called `.env` and update it with your organisation id and the API token you generated. This file is in .gitignore and will not be committed.

```bash
AXONOPS_ORG_ID="youraxonopsorg"
AXONOPS_API_SECRET_TOKEN="yourapitoken"
```

For self-hosted AxonOps users you can also add a `AXONOPS_DASH_URL` variable to point at your own installation of AxonOps.

#### **4. Set Up a Python Virtual Environment**

Create a virtual environment in the project directory using `.venv` as the directory name. This ensures that all Python dependencies are installed in an isolated environment specific to this project:

```bash
python3 -m venv .venv
```

This command creates a directory named `.venv` inside your project directory, which contains the virtual environment.

#### **5. Activate the Virtual Environment**

Activate the virtual environment to start using it. The activation command depends on your operating system:

- **On macOS and Linux:**

  ```bash
  source .venv/bin/activate
  ```

- **On Windows (Command Prompt):**

  ```cmd
  .venv\Scripts\activate.bat
  ```

- **On Windows (PowerShell):**

  ```powershell
  .venv\Scripts\Activate.ps1
  ```

Once activated, your terminal prompt should change to indicate that you are working within the virtual environment.

#### **6. Install Required Python Packages**

With the virtual environment activated, install all necessary packages listed in `requirements.txt`:

```bash
pip install -r requirements.txt
```

This command reads the `requirements.txt` file and installs all specified packages into the virtual environment.

#### **7. Run the Python Script**

Now that all dependencies are installed, you can run the main Python script:

```bash
python axonops_csv_extractor.py [options]
```

Usage:
```bash
usage: axonops_csv_extractor.py [-h] -o OUTPUTDIR -q QUERYCONFIG -m MONTHOFYEAR [-d]

AxonOps CSV Extractor

options:
  -h, --help            show this help message and exit
  -o OUTPUTDIR, --outputdir OUTPUTDIR
                        The file path to a directory for outputting the CSV data.
  -q QUERYCONFIG, --queryconfig QUERYCONFIG
                        File path to the JSON configuration file listing the queries to run and extract to CSV. See the README.md for more information on this configuration file.
  -m MONTHOFYEAR, --monthofyear MONTHOFYEAR
                        The month of year in format YYYYMM for which data will be extracted to CSV. This can not be in the future nor the current month
  -d, --deletejson      If set, the downloaded JSON will be kept in the output directory. By default it is automatically deleted after being converted to CSV.
```


#### **8. Deactivate the Virtual Environment**

Once you are done working, deactivate the virtual environment to return to your system's default Python environment:

```bash
deactivate
```

***

*AxonOps is a registered trademark of AxonOps Limited. Apache, Apache Cassandra, Cassandra, Apache Spark, Spark, Apache TinkerPop, TinkerPop, Apache Kafka and Kafka are either registered trademarks or trademarks of the Apache Software Foundation or its subsidiaries in Canada, the United States and/or other countries.*
