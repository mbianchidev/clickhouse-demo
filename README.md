# ClickHouse Demo

This is a sample clickhouse Demo project. 
It demonstrates how to use ClickHouse with a simple example with docker-compose and on Kubernetes.

Follow through with [this presentation](https://docs.google.com/presentation/d/1kLC60nBMNH2SHYAo09MtEHgNK779K7vk_QXMW3zecOE/edit#slide=id.p1).

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

### Testing ClickHouse

Connect to the ClickHouse server using the installed client

```bash
# it connects by default to localhost:9000
./clickhouse client 

# or you can specify the host and port
./clickhouse client --host localhost --port 9000
```

[Create a table](https://clickhouse.com/docs/en/sql-reference/statements/create/table) in ClickHouse, for example:

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

For more info about the MergeTree table engine, check the [documentation](https://clickhouse.com/docs/en/engines/table-engines/mergetree-family/mergetree#:~:text=The%20MergeTree%20engine%20and%20other,rates%20and%20huge%20data%20volumes.)

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

Some aggregate queries you can run 1-by-1:

```sql
SELECT avg(salary) FROM random_data;
SELECT max(age) FROM random_data;
SELECT count() FROM random_data;
```

For a full reference of the SQL syntax, check the [documentation](https://clickhouse.com/docs/en/sql-reference).

### Running ClickHouse on Kubernetes - operator

First, you need to create a Kubernetes cluster. 

You can use Minikube or k3s locally, or any other Kubernetes cluster in the cloud like EKS, AKS or GKE. 

In our case, we will use a pre-existing Kubernetes cluster offered by Civo Cloud, see this [readme](/kubernetes/civo.md) and the Civo [documentation website](https://www.civo.com/docs/account/signing-up) for more information.

Now, you can deploy ClickHouse on Kubernetes as follows:

```bash
# get access to your k8s cluster (always remember to switch context!)
kubectl apply -f operator/operator.yaml -n kube-system

kubectl get pods -A | grep clickhouse-operator

kubectl create namespace test-clickhouse-operator

kubectl apply -n test-clickhouse-operator -f operator/example-1.yaml

kubectl -n test-clickhouse-operator exec -it chi-simple-01-simple-0-0-0 -- clickhouse-client
```

Now you can create a table and insert some data as we did before.
This example doesn't use any service to connect externally or permanent storage, so the data will be lost when the pod is deleted.

To clean up

```bash
kubectl delete -n test-clickhouse-operator -f operator/example-1.yaml

kubectl delete namespace test-clickhouse-operator

kubectl delete -f operator/operator.yaml -n kube-system

kubectl get pods -A | grep clickhouse-operator
```

### Running ClickHouse on Kubernetes - vanilla

Same as before but, after getting the access to the cluster you'll need to run the following commands:

```bash
# get access to your k8s cluster (always remember to switch context!)
kubectl create namespace clickhouse
kubectl apply -f pv.yaml -n clickhouse
kubectl apply -f pvc.yaml -n clickhouse
kubectl apply -f config.yaml -n clickhouse
kubectl apply -f deployment.yaml -n clickhouse
kubectl apply -f svc.yaml -n clickhouse
```

Now verify that the ClickHouse server is running on Kubernetes

```bash
kubectl get pods -n clickhouse
```

Once the ClickHouse pod is running, you can connect to it using the ClickHouse client

From within the Kubernetes cluster:

```bash
kubectl exec -it clickhouse-client -n clickhouse -- clickhouse client --host clickhouse-server
```

With port-forwarding:

```bash
# Forward the port 9000 to your local machine to use the CLI
kubectl port-forward svc/clickhouse 9000:9000 -n clickhouse
./clickhouse client --host localhost --port 9000
# Forward the port 8123 to use the HTTP interface
kubectl port-forward svc/clickhouse 8123:8123 -n clickhouse
# visit https://localhost:8123/play
```

From the load balancer:

```bash
# Get the external IP of the load balancer and connect to it
./clickhouse client --host $(kubectl get svc clickhouse -n clickhouse -o jsonpath='{.status.loadBalancer.ingress[0].hostname}') --port 9000
```

Now you can create a table and insert some data as we did before.
Happy querying!

### Cleaning up

To clean up the resources created by ClickHouse on Kubernetes, you can run the following commands:

```bash
kubectl delete -f svc.yaml -n clickhouse
kubectl delete -f deployment.yaml -n clickhouse
kubectl delete -f config.yaml -n clickhouse
kubectl delete -f pvc.yaml -n clickhouse
kubectl delete -f pv.yaml -n clickhouse
kubectl delete namespace clickhouse
```
