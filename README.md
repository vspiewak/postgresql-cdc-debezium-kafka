# PostgreSQL CDC Debezium Kafka

Read more here : https://vspiewak.com/turn-postgresql-to-event-stream-with-kafka-connect-and-debezium


## Starting services
    
    docker compose up -d


## Setup PostgreSQL

    # show current wal_level, default 'replica'
    PGPASSWORD=my_password docker exec -it postgres psql my_db -U my_user -c "SHOW wal_level"

    wal_level
    -----------
    replica
    (1 row)

    # set wal_level to logical using SQL...
    PGPASSWORD=my_password docker exec -it postgres psql my_db -U my_user -c "ALTER SYSTEM SET wal_level = logical;"

    # ... or directly in postgresql.conf
    docker exec postgres bash -c "echo 'wal_level = logical' >> /var/lib/postgresql/data/postgresql.conf"

    # restart container
    docker restart postgres

    # wal_level is now 'logical'
    PGPASSWORD=my_password docker exec -it postgres psql my_db -U my_user -c "SHOW wal_level"

    wal_level
    -----------
    logical
    (1 row)


## Register Debezium PostgreSQL connector

    curl -i -X POST \
    -H "Accept:application/json" \
    -H  "Content-Type:application/json" \
    http://localhost:8083/connectors -d '
    {
    "name": "cdc-connector",
    "config": {
        "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
        "topic.prefix": "cdc",
        "database.hostname": "postgres",
        "database.port": "5432",
        "database.user": "my_user",
        "database.password": "my_password",
        "database.dbname" : "my_db",
        "plugin.name": "pgoutput"
    }
    }'

    HTTP/1.1 201 Created
    Date: Mon, 14 Oct 2024 19:30:16 GMT
    Location: http://localhost:8083/connectors/cdc-connector
    Content-Type: application/json
    Content-Length: 342
    Server: Jetty(9.4.54.v20240208)

    {"name":"cdc-connector","config":{"connector.class":"io.debezium.connector.postgresql.PostgresConnector","topic.prefix":"cdc","database.hostname":"postgres","database.port":"5432","database.user":"my_user","database.password":"my_password","database.dbname":"my_db","plugin.name":"pgoutput","name":"cdc-connector"},"tasks":[],"type":"source"}%


## Some SQL actions

    # launch psql
    PGPASSWORD=my_password docker exec -it postgres psql my_db -U my_user
    
    # create table user
    CREATE TABLE users (
    id  char(10) NOT  NULL,
    firstname char(50),
    lastname char(50),
    PRIMARY KEY (id)
    );

    # insert a row
    INSERT INTO users (id, firstname, lastname) VALUES('id1', 'Vincent', 'Spiewak');

    # update a row
    UPDATE users SET firstname='Vince' where id='id1';

    # delete a row
    DELETE FROM users WHERE id='id1';