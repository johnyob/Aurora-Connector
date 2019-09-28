# AWS Aurora Serverless Connector
> This is still very much in development.

The Aurora Serverless Connector is a wrapper that provides a *nice* interface with the Amazon Aurora Serverless
Data API by abstracting away the notion of pagination, transactions and field values. 



## Simple Examples

The Aurora Serverless Connector makes working with the Aurora Serverless Data API almost as
simple as using the MySQL Connector. First we must instantiate the `Database` interface with some basic 
configuration information, and then use the various methods to retrieve data from our Aurora Serverless Cluster.

```python
from aurora_connector.Database import AuroraDatabase

AURORA_DATABASE = AuroraDatabase(
    database_name="MyDatabase",
    db_cluster_arn="arn:aws:rds:eu-west-1:XXXXXXXXXXXX:cluster:my-cluster",
    db_credentials_secrets_store_arn="arn:aws:secretsmanager:eu-west-1:XXXXXXXXXXXX:secret:my-db-secret"
)

# Fetch all users from User table.
result = AURORA_DATABASE.query_fetch_all(
    "SELECT * FROM User;"
)

# Fetch the first 10 users from User table.
result_page = AURORA_DATABASE.query(
    "SELECT * FROM User;",
    page=1,
    per_page=10
)

# Fetch the first 10 users from User table where name = "Alistair"
result_page_and_params = AURORA_DATABASE.query(
    "SELECT * FROM User WHERE name = :name;",
    sql_parameters={"name": "Alistair"},
    page=1,
    per_page=10
)
```

The Connector also provides automatic transaction creation and management. We using any method that wraps the
`execute` method, then a new transaction is created. Any subsequent statements will executed as part of the transaction
until the `commit` or `rollback` method is called. 
 
For example:
```python
from aurora_connector.Database import AuroraDatabase, AuroraDatabaseException

AURORA_DATABASE = AuroraDatabase(
    database_name="MyDatabase",
    db_cluster_arn="arn:aws:rds:eu-west-1:XXXXXXXXXXXX:cluster:my-cluster",
    db_credentials_secrets_store_arn="arn:aws:secretsmanager:eu-west-1:XXXXXXXXXXXX:secret:my-db-secret"
)

try:
    AURORA_DATABASE.insert(
        "INSERT INTO User (username, name) VALUES (:username, :name);",
        sql_parameters={
            "username": "johnyob",
            "name": "Alistair O'Brien"
        }
    )
    
    AURORA_DATABASE.commit()
except AuroraDatabaseException as error:
    AURORA_DATABASE.rollback()
```

## Usage

The Aurora Serverless Connector simply wraps the RDSDataService client (from boto3), providing solutions and a
range of convenient features to fix the Data API's 'strangeness' and make writing applications that use the 
Aurora Serverless service easier. 

In order to use the connector, it must first be instantiated with the required configuration:

| Property                         | Type   | Description                                                 |
|----------------------------------|--------|-------------------------------------------------------------|
| database_name                    | String | The database to use with queries and other SQL statements.  |
| db_cluster_arn                   | String | The ARN of the Aurora Serverless Cluster                    |
| db_credentials_secrets_store_arn | String | The ARN of the secret that stores the database credentials. |


### Queries

Once instantiated, running a query is extremely easy. Use the `query` method to get a paginated result set with 
a maximum of 1000 records per page, or `query_fetch_all` to fetch all records in the result set.

```python
result = AURORA_DATABASE.query("SELECT username, name FROM User;")
```
This will return an array of arrays, representing a list of records
```
[
    ["johnyob", "Alistair"],
    ["username1", "John"],
    .
    .
    .
]
```

When querying with parameters, we can used named parameters in our SQL statements, and then provide a dictionary
that maps parameter names to values. For example:
```python
result = AURORA_DATABASE.query(
    "SELECT username, name FROM User WHERE username = :username;", 
    sql_parameters={"username": "johnyob"}
)
```
returns
```
[
    ["johnyob", "Alistair"]
]
```

### Transactions

Transaction support in Aurora Serverless Connector is almost automatic. We start a new transaction (if one doesn't already exist)
when using a method that wraps the `execute` method from the `AuroraDatabase` class. This includes:
- `insert`
- `delete`
- `update`

If an error occurs during an Aurora Serverless Connector operation, a `AuroraDatabaseException` is thrown. So we can use
try / except blocks and the methods `commit` and `rollback` to provide an "all-or-nothing" proposition for the Aurora
Serverless statements. 

For example:
```python
try:
    AURORA_DATABASE.delete(
        "DELETE FROM User WHERE username = :username;",
        sql_parameters={"username": "johnyob"}
    )
    
    AURORA_DATABASE.commit()
except AuroraDatabaseException as error:
    AURORA_DATABASE.rollback()
```

## Contributions

Any contributions, ideas or bug reports are welcome and extremely appreciated. 

## Credits

- [Alistair O'Brien](https://github.com/johnyob)