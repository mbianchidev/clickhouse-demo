# ClickHouse Demo

This is a sample clickhouse Demo project. 
It demonstrates how to use ClickHouse with a simple example with docker-compose and on Kubernetes.

## Prerequisites

- Docker
- Docker Compose
- A Kubernetes cluster running somewhere
- kubectl

## Getting Started

Install ClickHouse client

```bash
curl https://clickhouse.com/ | sh
```

### Running ClickHouse locally on Docker with docker-compose

Now, you can run the ClickHouse server using docker-compose

```bash
cd docker && docker-compose up
```

### Testing ClickHouse locally

Connect to the ClickHouse server using the installed client

```bash
# it connects by default to localhost:9000
./clickhouse client 

# or you can specify the host and port
./clickhouse client --host localhost --port 9000
```

Create a table in ClickHouse

```sql
CREATE TABLE random_data (
    id UInt32,
    name String,
    age UInt8,
    salary Float32,
    join_date Date
) ENGINE = MergeTree()
ORDER BY id;
```

Now let's insert some data!

```sql
INSERT INTO random_data
SELECT
    number AS id,
    concat('John_Doe_', toString(number)) AS name,
    rand() % 42 AS age,
    round(rand() % 100000 / 10.0, 2) AS salary,
    toDate('2024-01-01') + number % 1000 AS join_date
FROM numbers(10000);
```

Query the data

```sql
SELECT * FROM random_data LIMIT 10;
```

Some aggregate queries

```sql
SELECT avg(salary) FROM random_data;
SELECT count() FROM random_data;
```

### Running ClickHouse on Kubernetes