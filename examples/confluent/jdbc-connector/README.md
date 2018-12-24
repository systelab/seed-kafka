# Stream data between 2 MySql databases

This example streams data from a [seed-jee](https://github.com/systelab/seed-jee) MySql database to Apache Kafka and from Kafka to another [seed-jee](https://github.com/systelab/seed-jee) MySql database without any kind of code using Confluent and JDBC Kafka Connect connectors.

After following next steps you can create or modify a patient with [seed-angular](https://github.com/systelab/seed-angular) connected to a seed-jee backend and instantly changes will be applied to the other seed-jee backend.

## 1. Environment startup

Execute [docker-compose.yml](docker-compose.yml) file:

```
docker-compose up -d
```

After executing the command the output should show:

```
Creating zookeeper                 ... done
Creating jdbc-connector_db2_1 ... done
Creating jdbc-connector_db1_1 ... done
Creating jdbc-connector_backend1_1 ... done
Creating jdbc-connector_backend2_1 ... done
Creating broker                    ... done
Creating schema-registry           ... done
Creating connect                   ... done
```

This yaml file starts up the following containers:

- MySql database 1
- seed-jee backend 1
- MySql databse 2
- seed-jee backend 2
- Apache Zookeper: Kakfa uses Zookeeper as a a centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services.
- Confluent Kafka: Docker image of kafka distributed with Confluent open source distribution
- Confluent schema registry: server that runs in the infrastructure (close to Kafka brokers) and that stores schemas (including all their versions). When sending Avro messages to Kafka, the messages contain an identifier of a schema stored in the Schema Registry. [More info](https://docs.confluent.io/current/schema-registry/docs/index.html)
- Confluent Kafka Connect: open source component of Apache Kafka, is a framework for connecting Kafka with external systems such as databases, key-value stores, search indexes, and file systems. Using Kafka Connect you can use existing connector implementations for common data sources and sinks to move data into and out of Kafka.

## 2. Configure and load JDBC connector/sink in Kafka Connect

This example uses Kafka Connect JDBC connector. Uses JDBC source connector to import data from MySql database 1 and 2 with a JDBC driver into Kafka topics. Uses the JDBC sink connector to export data from Kafka topics to MySql datbase 1 and 2 with a JDBC driver.

In this folder there are four files with source and sink configuration:

- [kafka-connect-jdbc-source-db1.json](kafka-connect-jdbc-source-db1.json)
- [kafka-connect-jdbc-source-db2.json](kafka-connect-jdbc-source-db2.json)
- [kafka-connect-jdbc-sink-db1.json](kafka-connect-jdbc-sink-db1.json)
- [kafka-connect-jdbc-sink-db2.json](kafka-connect-jdbc-sink-db2.json)

In this example source connector is configured to observe changes and new rows in table patient. When a patient is created or modified the connector will send a new message to a kafka topic. To use this configuration a timestamp column with update time and a column with unique id is needed.

Source configuration properties used in this example:

| Name | Value | Description |
| ---- |:----------:| ------------|
| connection.url | jdbc:mysql://db1:3306/SEED | jdbc url database connection |
| connection.user | SEED | database username |
| connection.password | SEED | database password |
| table.whitelist | patient | table with data to send to kafka |
| mode | timestamp | timestamp column that detects new and modified rows |
| timestamp.column.name | updateTime | column that has the timestamp value to use |
| topic.prefix | seed-jee-mysql-db1- | Kafka topic will have this prefix and the table name|

You can find more information about source connector configuration properties [here](https://docs.confluent.io/current/connect/kafka-connect-jdbc/source-connector/source_config_options.html).

Also a sink connector is configured to receive message from Kafka topic with new patients or patients modifications. When a new message is received from the topic the sink connector creates the new patient or updates the changes sent.

Source configuration properties used in this example:

| Name | Value | Description |
| ---- |:----------:| ------------|
| connection.url | jdbc:mysql://db2:3306/SEED | jdbc url database connection |
| connection.user | SEED | database username |
| connection.password | SEED | database password |
| topics | seed-jee-mysql-db1-patient | The name of the Kafka topic with the table to insert or update |
| insert.mode | upsert | Use the appropriate upsert (update and insert) semantics for the target database if it is supported by the connector. |
| table.name.format | patient | Name of the table to insert or update |
| pk.mode | record_value | primary key mode, in this case fields from the record value are used |
| pk.fields | id | columns with the pk |

You can find more information about sink connector configuration properties [here](https://docs.confluent.io/current/connect/kafka-connect-jdbc/sink-connector/sink_config_options.html)

One source and one sink is configured for each database, one for send messages to the other database and one for receive messages from the other database.

To load these configuration files and create the topics you need to execute the following commands in the command line:

```
curl -X POST -H "Content-Type: application/json" --data "@kafka-connect-jdbc-source-db1.json" http://localhost:8083/connectors
curl -X POST -H "Content-Type: application/json" --data "@kafka-connect-jdbc-source-db2.json" http://localhost:8083/connectors
curl -X POST -H "Content-Type: application/json" --data "@kafka-connect-jdbc-sink-db1.json" http://localhost:8083/connectors
curl -X POST -H "Content-Type: application/json" --data "@kafka-connect-jdbc-sink-db2.json" http://localhost:8083/connectors
```


You can verify the status of each source and sink connector executing the following commands:

```
curl -s -X GET http://localhost:8083/connectors/jdbc_source_mysql_seed_jee_db1/status
curl -s -X GET http://localhost:8083/connectors/jdbc_source_mysql_seed_jee_db2/status
curl -s -X GET http://localhost:8083/connectors/jdbc_sink_mysql_seed_jee_db1/status
curl -s -X GET http://localhost:8083/connectors/jdbc_sink_mysql_seed_jee_db2/status
```

Now you can execute two [seed-angular](https://github.com/systelab/seed-angular) applications connected to the seed-jee backends (ports 8080 and 9080). Try to create/modify a patient from first application and changes will be available in the second application instantly.