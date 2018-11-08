## Preliminaries
Step by step tutorial : https://redash.io/help/open-source/dev-guide/setup
### Redis server
Firstly, install the [`redis`](https://linode.com/docs/databases/redis/install-and-configure-redis-on-centos-7/) server. If OS is CentOS, type the following command
```
sudo yum install redis
```

Next, run the `redis` server using the following command
```
sudo systemctl start redis
```

(**OPTIONAL**) To start `redis` on boot
```
sudo systemctl enable redis
```

To check whether the server has been correctly start up, type the following command
```
redis-cli ping
```
If the terminal output `PONG`, redis server is up and running.

### PostgreSQL
Installation steps depend heavily on your OS. In the steps below, commands showed are only applicable to CentOS7. For more information refers to [PostgreSQL Official Site](https://www.postgresql.org/download/linux/redhat/)

Firstly, install the repository so that the package is visible to our `yum` package manager
```
yum install https://download.postgresql.org/pub/repos/yum/11/redhat/rhel-7-x86_64/pgdg-centos11-11-2.noarch.rpm
```

Next, proceed to install `postgresql11` and its server counterparts
```
yum install postgresql11
yum install postgresql11-server
```

Then, initialize the database as well as start running it.
```
/usr/pgsql-11/bin/postgresql-11-setup initdb  
systemctl enable postgresql-11  
systemctl start postgresql-11
```

## Installting ReDASH

1. FOLLOWS STEPS
 https://redash.io/help/open-source/dev-guide/setup

BLA

Create db table 
```
bin/run ./manage.py database create_tables
```

RUNNING THE SERVICES

1. Web server:  `bin/run ./manage.py runserver --debugger --reload`
2. Celery:  `./bin/run celery worker --app=redash.worker --beat -Qscheduled_queries,queries,celery -c2`
3. Webpack dev server:  `npm run start`

To start Web server at '0.0.0.0', simply add the `--host` command line argument as shown below
`bin/run ./manage.py runserver --debugger --reload --host 0.0.0.0`

## Issues
### 1. Error 111 connecting to localhost:6379. Connection refused. 
When this issue shows up, it means the `redis` server is not installed and/or not running. Refers to **Preliminaries** step in this document to see how to set up properly.

### 2. sqlalchemy.exc.OperationalError: (psycopg2.OperationalError) could not connect to server: No such file or directory. Is the server running locally and accepting connections on Unix domain socket "/var/run/postgresql/.s.PGSQL.5432"? 
When this issue shows up, it means the `PostgreSQL` server is not installed and/or not running. Refers to **Preliminaries** step in this document to see how to set up properly.
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEwNjUzNjk3MTBdfQ==
-->