# Example Confluent JDBC Sink Connect
This quickstart example let's show how to use confluent JDBC sink Connect to export data from Kafka topics


## Requirements

* Confluent Platform
* Docker Compose


## Quickstart

To see the basic functionality of the connector, we’ll be copying data from a single topic to a local PostgreSQL database.

Let's see a configuration for the connector with following settings:

```properties
name=postgres-sink
connector.class=io.confluent.connect.jdbc.JdbcSinkConnector
tasks.max=1
topics=person
connection.url=jdbc:postgresql://localhost:5432/test_connect
connection.user=test_connect
connection.password=test_connect
auto.create=true
```

The first few settings are common settings for all connectors, except for `topics` which is specific to sink connectors like this one. The `connection.url`, `connection.user` and `connection.password` specifies the database configurations that will be used in connection.

 If `auto.create` is enabled, the connector can create the destination table if it not exists.


### Commands

We will use confluent API to setup our Postgres Sink connector. This example assumes you are running Kafka and Schema Registry locally on the default ports.

1. Start up Confluent Platform

```
confluent local start
```

2. Start Postgres using docker and `docker-compose.yml` from this project

Compose file:

```yml
version: '3.3'
services:
  postgres:
    image: postgres:9.6
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_DB=test_connect
      - POSTGRES_USER=test_connect
      - POSTGRES_PASSWORD=test_connect
      - MAX_CONNECTIONS=300
```

Start compose:

```
docker-compose up
```

3. Create JDBC PostgreSQL sink Connector using Kafka Connect REST Interface

```shell
curl --location --request POST 'http://localhost:8083/connectors' \
--header 'Content-Type: application/json' \
--data-raw '{
    "name": "postgres-sink",
    "config": {
        "connector.class": "io.confluent.connect.jdbc.JdbcSinkConnector",
        "tasks.max": "1",
        "topics": "person",
        "connection.url": "jdbc:postgresql://localhost:5432/test_connect",
        "connection.user": "test_connect",
        "connection.password": "test_connect",
        "auto.create": true
    }
}'
```

4. Sending messages to a topic

Message Payload

```json
{
    "id": 1,
    "full_name": "test"
}
```

```shell
kafka-avro-console-producer \
 --broker-list localhost:9092 --topic person \
 --property value.schema='{"type":"record","name":"person","fields":[{"name":"id","type":"int"},{"name":"full_name","type":"string"}]}'
```

After previous command, console wait any entries like that:

```json
{"id": 1, "full_name": "test"}
```

5. Execute select in postgres

```sql
select * from person;
```

Results:

| id       |     full_name | 
|----------|:-------------:|
| 1        |  test         |


## References

* [Confluent Platform](https://docs.confluent.io/current/quickstart/ce-quickstart.html#ce-quickstart)
* [JDBC Sink Connect](https://docs.confluent.io/3.2.2/connect/connect-jdbc/docs/sink_connector.html#quickstart)
* [Kafka Connect REST Interface](https://docs.confluent.io/current/connect/references/restapi.html)



