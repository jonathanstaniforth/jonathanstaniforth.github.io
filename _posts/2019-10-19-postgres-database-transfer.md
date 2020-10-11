---
layout: post
title:  "Postgres Transfer Between Docker Containers"
date:   2019-10-19 18:00:00 +0800
tags: [docker,postgres,database]
navigation: true
---
This post provides a brief guide to transferring a PostgreSQL database from one Docker container to another, when they reside on different servers.

## Origin Server
Starting at the origin server, i.e. the server where the current PostgreSQL database is running, dump the database to an sql file.

```bash
pg_dump -U <database_username> <database_name> > postgres_backup.sql
```

Once the backup has complete, transfer the sql file from the origin server to the destination server, i.e. the server where the new PostgreSQL database will run.

```bash
scp -i <location_of_private_ssh_key> postgres_backup.sql <username>@<destination_server_private_ip>:~
```

> Make sure to have a private SSH key on the origin server and the matching public SSH key on the destination server.

## Destination Server

At the destination server, login and change the owner of the sql file to **postgres**. If necessary, move the sql file to the folder which will contain the database files.

```bash
chown postgres:postgres <sql_file_path>
```

Now, pull the postgres Docker image:

```bash
docker pull postgres:<tag>
```

Next, run the postgres image, mounting the folder which will contain the database files:

```bash
docker run -d -v <database_folder_path>:/var/lib/postgresql/data postgres:<tag>
```

Find the running container, enter and navigate to the database folder:

```bash
# Get the container ID
docker container ls

docker exec -it <container_id> bash

cd /var/lib/postgresql/data
```

Now, run the following commands as postgres commands:

```sql
CREATE USER <database_username> WITH PASSWORD '<password>';
GRANT ALL PRIVILEGES ON DATABASE <database_name> TO <database_username>;
```

Finally, import the sql file into the new database:

```bash
psql -U <database_username> <database_name> < postgres_backup.sql
```

> Make sure no errors appear in the output while importing the data into the database.