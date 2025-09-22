# Create a Docker Container for Sql Server with a Sql Backup
Simple walkthrough to create a docker container that restores a database from a backup. Useful for more realistic debugging for development, testing, QA.
Seeding a database is good for minor tests but using a backup with a data set representative of actual data is much more effective for quality testing.

## Create the restore script, restore-DatabaseName.sql; replace DatabaseName with the database name
```
RESTORE DATABASE [DatabaseName]
FROM DISK = 'tmp/DatabaseName.bak'
WITH MOVE 'DatabaseName' TO '/var/opt/mssql/data/DatabaseName.mdf',
MOVE 'DatabaseName_Log' TO '/var/opt/mssql/data/DatabaseName_log.ldf',
REPLACE;
GO
```

## Create the Dockerfile
```
FROM mcr.microsoft.com/mssql/server:2022-latest AS build

ENV ACCEPT_EULA=Y
ENV SA_PASSWORD="P@ssWord!!2"

WORKDIR /tmp

COPY DatabaseName.bak .
COPY restore-DatabaseName.sql .

RUN ( /opt/mssql/bin/sqlservr & ) | grep -q "Service Broker manager has started" \
    && sleep 5 \
    && /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P "P@ssWord!!2" -i /tmp/restore-DatabaseName.sql \
    && pkill sqlservr

FROM mcr.microsoft.com/mssql/server:2022-latest AS release

ENV ACCEPT_EULA=Y
COPY --from=build /var/opt/mssql/data /var/opt/mssql/data
```
## Build the docker image
```
docker build -t projectName-sql-dev-image .
```
## Run docker container
```
docker run -d -p 1433:1433 --name projectName-sql-dev-container projectName-sql-dev-image
```
## Connect to localhost from SSMS
Connect to the container from SSMS to verify the database. Note, if the host port is modified, use localhost, port-number to connect
