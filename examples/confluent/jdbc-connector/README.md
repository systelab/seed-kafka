curl -X POST -H "Content-Type: application/json" --data "@kafka-connect-jdbc-source-db1.json" http://localhost:8083/connectors
curl -X POST -H "Content-Type: application/json" --data "@kafka-connect-jdbc-source-db2.json" http://localhost:8083/connectors
curl -X POST -H "Content-Type: application/json" --data "@kafka-connect-jdbc-sink-db1.json" http://localhost:8083/connectors
curl -X POST -H "Content-Type: application/json" --data "@kafka-connect-jdbc-sink-db2.json" http://localhost:8083/connectors

curl -s -X GET http://localhost:8083/connectors/jdbc_source_mysql_seed_jee_db1/status
curl -s -X GET http://localhost:8083/connectors/jdbc_source_mysql_seed_jee_db2/status
curl -s -X GET http://localhost:8083/connectors/jdbc_sink_mysql_seed_jee_db1/status
curl -s -X GET http://localhost:8083/connectors/jdbc_sink_mysql_seed_jee_db2/status

docker run -d -e BACKEND='http://localhost:8080' -p 9081:80 systelab/seed-angular
docker run -d -e BACKEND='http://localhost:9080' -p 10081:80 systelab/seed-angular