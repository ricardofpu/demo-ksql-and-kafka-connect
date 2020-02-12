# Example Confluent JDBC Sink Connect
This quickstart example let's show how to use confluent JDBC sink Connect to export data from Kafka topics


## Requirements

* Confluent Platform
* Docker Compose


## Quickstart

To see the basic functionality of the connector, we’ll be copying data from a single topic to a local PostgreSQL database.

Let's create a configuration file for the connector with following settings:

```properties
name=postgres-sink
connector.class=io.confluent.connect.jdbc.JdbcSinkConnector
tasks.max=1
topics=nickname
connection.url=jdbc:postgresql://localhost:5432/test_connect
connection.user=test_connect
connection.password=test_connect
auto.create=true
```

The first few settings are common settings for all connectors, except for `topics` which is specific to sink connectors like this one. The `connection.url` specifies the database that will be used in connection. If `auto.create` is enabled, the connector can create the destination table if it not exists.


## Confluent connector

We will use confluent API to setup our Postgres Sink connector. This example assumes you are running Kafka and Schema Registry locally on the default ports.

### Commands

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

```console
curl --location --request PUT 'http://localhost:8083/connectors/postgres-sink/config' \
--header 'Content-Type: application/json' \
--data-raw '{
	"connector.class": "io.confluent.connect.jdbc.JdbcSinkConnector",
	"tasks.max": "1",
	"topics": "nickname",
	"connection.url": "jdbc:postgresql://localhost:5432/test_connect",
	"connection.user": "test_connect",
	"connection.password": "test_connect",
	"auto.create": true
}'
```
> Reference: https://docs.confluent.io/current/connect/references/restapi.html


4. Sending messages to a topic (`nickname`)

Message Payload

```json
{
    "id": 1,
    "nickname": "test"
}
```

```console
kafka-avro-console-producer \
 --broker-list localhost:9092 --topic nickname \
 --property value.schema='{"type":"record","name":"myrecor","fields":[{"name":"id","type":"int"},{"name":"nickname","type":"string"}]}'
```

After previous command, console wait any entries like that:

```json
{"id": 1, "nickname": "test"}
```

5. Execute select in postgres

```sql
select * from nickname;
```

Results:

| id       |      nickname | 
|----------|:-------------:|
| 1        |  test         |


## References

* Download and start Confluent Platform: https://docs.confluent.io/current/quickstart/ce-quickstart.html#ce-quickstart
* JDBC sink Connect: https://docs.confluent.io/3.2.2/connect/connect-jdbc/docs/sink_connector.html#quickstart



