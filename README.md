# Astro Pipeline(Data Engineering Project): Pipeline built using Python, SQL, Astro SDK CLI, S3, Snowflake, Docker, and Apache Airflow
**What is Astro Pipeline?** Astro Pipeline is a pipeline built using Python, SQL, Astro SDK CLI, S3, Snowflake, Docker, and Apache Airflow for streamlined data extraction, transformation, and loading operations

## Tech Stack/Skils Used
1. Python
2. SQL
3. Astro SDK CLI
4. AWS S3
5. Snowflake
6. Docker
7. Apache Airflow

## Pipeline Architecture

The architecture diagram created below highlights and breaks down project:

![Screenshot 2023-06-19 at 5 45 09 PM](https://github.com/anujgarlapati/Astro-Pipeline/assets/59670482/af6eaa54-b3c5-47a8-bbae-3f9c6eef55e7)


## Airflow DAG

The following DAG in Airflow is represented with the following code: 

```
from datetime import datetime 

from airflow.models import DAG
from pandas import DataFrame

from astro import sql as aql 
from astro.files import File
from astro.sql.table import Table

S3_FILEPATH = "s3://anuj-astrosdk"
S3_CONN_ID ="aws_default"
SNOWFLAKE_CONN_ID = "snowflake_default"
SNOWFLAKE_ORDERS = "orders_table"
SNOWFLAKE_FILTERED_ORDERS = "filtered_table"
SNOWFLAKE_JOINED = "joined_table"
SNOWFLAKE_CUSTOMERS = "customers_table"
SNOWFLAKE_REPORTING = "reporting_table"

@aql.transform
def filter_orders(input_table: Table):
    return "SELECT * FROM {{input_table}} WHERE amount > 150"

@aql.transform
def join_orders_customers(filtered_orders_table: Table, customers_table:Table):
    return """ SELECT c.customer_id, customer_name, order_id, purchase_date, amount,
    type
    FROM {{filtered_orders_table}} f JOIN {{customers_table}} c
    on f.customer_id = c.customer_id """

@aql.dataframe
def transform_dataframe(df: DataFrame):
    purchase_dates = df.loc[: "purchase_date"]
    print("purchase dates:", purchase_dates)
    return purchase_dates


with DAG(dag_id='astro_orders', start_date=datetime(2023, 6 , 10), schedule='@daily', catchup=False):
    orders_data = aql.load_file(
        input_file =File(
            path = S3_FILEPATH + "/orders_data_header.csv", conn_id =S3_CONN_ID
        ),
        output_table=Table(conn_id=SNOWFLAKE_CONN_ID)
    )

    customers_table = Table(
        name=SNOWFLAKE_CUSTOMERS,
        conn_id = SNOWFLAKE_CONN_ID,

    )

    joined_data = join_orders_customers(filter_orders(orders_data), customers_table)

    reporting_table = aql.merge(
        target_table = Table(
            name=SNOWFLAKE_REPORTING,
            conn_id=SNOWFLAKE_CONN_ID,
        ),
        source_table = joined_data,
        target_conflict_columns=["order_id"],
        columns=["customer_id", "customer_name"],
        if_conflicts="update", 
        )
    
    purchase_dates = transform_dataframe(reporting_table)

    purchase_dates >> aql.cleanup()


```

The graph representation is as follows: 

![Screenshot 2023-06-19 at 5 59 18 PM](https://github.com/anujgarlapati/Astro-Pipeline/assets/59670482/6cf0afdb-8812-4340-95bd-8cf39eddce72)


## Conclusion

This project taught me how to create an efficent pipeline using Apache Airflow and Snowflake's data warehouse.

**Note:** The DAG file known as Astro_orders and the YAML file for the Airflow Settings are both attached. Additionally, the CSV file is also attached.
