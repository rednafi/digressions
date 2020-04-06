---
title: Up and Running with MySQL in Docker
toc: true
comments: true
layout: post
description: Setting up MySQL on Linux in no time
categories: [Database]
---

![Example image](https://images.unsplash.com/photo-1544383835-bda2bc66a55d?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1921&q=80)


## Setting Up

### Installation

This part describes the basic installation steps of setting up MySQL 5.7 server on Ubuntu Linux using docker.

* Install docker on your Linux machine. See the instruction [here](https://docs.docker.com/install/linux/docker-ce/ubuntu/).

* Install docker compose via following the instructions [here](https://docs.docker.com/compose/install/).

* Create another folder on your project folder and make a `docker-compose.yml` file. Run the following instructions one by one:

    ```bash
    mkdir mysql_dump
    cd mysql_dump
    touch docker-compose.yml
    ```

* Open the `docker-compose.yml` file and copy the following lines into it.

    ```yml
     # docker-compose version
    version: "3.3"

    services:
     # images
    mysql-dev:
        image: mysql:5.7
        environment:
        MYSQL_ROOT_PASSWORD: password
        MYSQL_DATABASE: test_db
        ports:
        - "3306:3306"
        # making data persistent
        volumes:
        - db-data:/var/lib/mysql

    volumes:
    db-data:
    ```

### Run MySQL Server

Run the `docker-compose` command. This will build and run the server in detached mode.
```bash
docker compose up -d
```

### Connect Shell to Server
Check the name of the running container with `docker ps` command. In this case, the running container is called `mysql_dumps_mysql-dev_1`. Then run the following command to connect your shell to the running server.

```bash
# connect shell to server
docker exec -it mysql_dumps_mysql-dev_1 mysql -uroot -p
```


### Alter Root Password

If you want to change the root password, enter the following command in the MySQL shell. Replace `MyNewPass` with your new root password:

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass';
```

You should see something like this in the command prompt:

```
Query OK, 0 rows affected (0.02 sec)
```

To make the change take effect, type the following command:

```sql
FLUSH PRIVILEGES;
```

### View Users

MySQL stores the user information in its own database. The name of the database is `mysql`. If you want to see what users are set up in the MySQL user table, run the following command:

```sql
SELECT User, Host, authentication_string FROM mysql.user;
```

You should see something like this:

```
+------------------+-----------+-------------------------------------------+
| User             | Host      | authentication_string                     |
+------------------+-----------+-------------------------------------------+
| root             | localhost | *2470C0C06DEE42FD1618BB99005ADCA2EC9D1E19 |
| mysql.session    | localhost | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE |
| mysql.sys        | localhost | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE |
| debian-sys-maint | localhost | *8282611144B9D51437F4A2285E00A86701BF9737 |
+------------------+-----------+-------------------------------------------+
4 rows in set (0.00 sec)
```

### Create a Database

According to the `docker-compose.yml` file, you already have created a database named `test_db`. You can create anotehr database named `test_db_2` via the following command:

```sql
CREATE DATABASE test_db_2;
```

List your databases via the following command:

```sql
SHOW DATABASES;
```

You should see something like this:

```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test_db            |
| test_db_2          |
+--------------------+
6 rows in set (0.01 sec)
```

To ensure the changes:

```sql
FLUSH PRIVILEGES;
```

### Creating Dummy Table in the Database

```sql
-- create dummy table
CREATE TABLE IF NOT EXISTS `student` (
  `id` int(2) NOT NULL DEFAULT '0',
  `name` varchar(50) CHARACTER SET utf8 NOT NULL DEFAULT '',
  `class` varchar(10) CHARACTER SET utf8 NOT NULL DEFAULT '',
  `mark` int(3) NOT NULL DEFAULT '0',
  `sex` varchar(6) CHARACTER SET utf8 NOT NULL DEFAULT 'male'
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

-- insert data into the dummy table
INSERT INTO `student` (`id`, `name`, `class`, `mark`, `sex`) VALUES
(1, 'John Deo', 'Four', 75, 'female'),
(2, 'Max Ruin', 'Three', 85, 'male'),
(3, 'Arnold', 'Three', 55, 'male'),
(4, 'Krish Star', 'Four', 60, 'female'),
(5, 'John Mike', 'Four', 60, 'female'),
(6, 'Alex John', 'Four', 55, 'male'),
(7, 'My John Rob', 'Fifth', 78, 'male'),
(8, 'Asruid', 'Five', 85, 'male'),
(9, 'Tes Qry', 'Six', 78, 'male'),
(10, 'Big John', 'Four', 55, 'female'),
(11, 'Ronald', 'Six', 89, 'female'),
(12, 'Recky', 'Six', 94, 'female'),
(13, 'Kty', 'Seven', 88, 'female'),
(14, 'Bigy', 'Seven', 88, 'female'),
(15, 'Tade Row', 'Four', 88, 'male'),
(16, 'Gimmy', 'Four', 88, 'male'),
(17, 'Tumyu', 'Six', 54, 'male'),
(18, 'Honny', 'Five', 75, 'male'),
(19, 'Tinny', 'Nine', 18, 'male'),
(20, 'Jackly', 'Nine', 65, 'female'),
(21, 'Babby John', 'Four', 69, 'female'),
(22, 'Reggid', 'Seven', 55, 'female'),
(23, 'Herod', 'Eight', 79, 'male'),
(24, 'Tiddy Now', 'Seven', 78, 'male'),
(25, 'Giff Tow', 'Seven', 88, 'male'),
(26, 'Crelea', 'Seven', 79, 'male'),
(27, 'Big Nose', 'Three', 81, 'female'),
(28, 'Rojj Base', 'Seven', 86, 'female'),
(29, 'Tess Played', 'Seven', 55, 'male'),
(30, 'Reppy Red', 'Six', 79, 'female'),
(31, 'Marry Toeey', 'Four', 88, 'male'),
(32, 'Binn Rott', 'Seven', 90, 'female'),
(33, 'Kenn Rein', 'Six', 96, 'female'),
(34, 'Gain Toe', 'Seven', 69, 'male'),
(35, 'Rows Noump', 'Six', 88, 'female');
```

### Show Tables

```sql
USE test_db;
SHOW tables;
```

### Delete a Database

To delete a database `test_db` run the following command:

```sql
DROP DATABASE test_db,

FLUSH PRIVILEGES;
```

### Add a Database User

To create a new user (here, we created a new user named `redowan` with the password `password`), run the following command in the MySQL shell:

```sql
CREATE USER 'redowan'@'localhost' IDENTIFIED BY 'password';

FlUSH PRIVILEGES;
```

Ensure that the changes has been saved via running `FLUSH PRIVILEGES;`. Verify that a user has been successfully created via running the previous command:

```sql
SELECT User, Host, authentication_string FROM mysql.user;
```

You should see something like below. Notice that a new user named `redowan` has been created:

```
+------------------+-----------+-------------------------------------------+
| User             | Host      | authentication_string                     |
+------------------+-----------+-------------------------------------------+
| root             | localhost | *2470C0C06DEE42FD1618BB99005ADCA2EC9D1E19 |
| mysql.session    | localhost | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE |
| mysql.sys        | localhost | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE |
| debian-sys-maint | localhost | *8282611144B9D51437F4A2285E00A86701BF9737 |
| redowan          | localhost | *0756A562377EDF6ED3AC45A00B356AAE6D3C6BB6 |
+------------------+-----------+-------------------------------------------+
```

### Delete a Database User

To delete a database user (here, I'm deleting the user-`redowan`) run:

```sql
DELETE FROM mysql.user
WHERE user='<redowan>'
AND host = 'localhost'

FlUSH PRIVILEGES;
```

### Grant Database User Permissions

Give the user full permissions for your new database by running the following command (Here, I provided full permission of `test_db` to the user `redowan`:

```sql
GRANT ALL PRIVILEGES ON test_db.table TO 'redowan'@'localhost';
```

If you want to give permission to all the databases, type:

```sql
GRANT ALL PRIVILEGES ON *.* TO 'redowan'@'localhost';

FlUSH PRIVILEGES;
```

### Loading Sample Database to Your Own MySQL Server

To load `mysqlsampledatabase.sql` to your own server (In this case the user is `redowan`. Provide database `password` in the prompt), first fireup the server and type the following commands:

```sql
mysql -u redowan -p test_db < mysqlsampledatabase.sql;
```

Now run:

```sql
SHOW DATABASES;
```
You should see something like this:

```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| classicmodels      |
| mysql              |
| performance_schema |
| sys                |
| test_db            |
+--------------------+
6 rows in set (0.00 sec)
```

### Stop the Server

The following command stops the server.

```bash
docker-compose down
```

Notice that a new database named `classicmodels` has been added to the list.


## Connecting to a Third Party Client

We will be using [DBeaver](https://github.com/dbeaver/dbeaver) as a third party client. While you can use the `mysql` shell to work on your data, a third partly client that make the experience much better with auto formatting, earsier import features, syntax highlighting etc.

### Installing DBeaver

You can install DBeaver installer from [here](https://dbeaver.io/download/). Installation is pretty straight forward.

### Connecting MySQL Database to DBeaver

Fire up DBeaver and you should be presented with this screen. Select `MySQL 8+` and go `next`.

![alt text](https://github.com/rednafi/brisk-SQL/raw/master/figs/fig1.png)

The dialogue box will ask for credentials to connect to a database. In this case, I will log into previously created local database `test_db` with the username `redowan`, corresponding password `password` and press `test connection` tab. A dialogue box might pop up, prompting you download necessary drivers.

![alt text](https://github.com/rednafi/brisk-SQL/raw/master/figs/fig2.png)

If everything is okay, you should see a success message. You can select the `SQL Editor` and start writing your MySQL scripts right away.

## Connecting to MySQL Server via Python

`PyMySQL` and `DBUtils` can be used to connect to MySQL Server.
```python
import pymysql
import os

from dotenv import load_dotenv
from DBUtils.PooledDB import PooledDB

load_dotenv(verbose=True)

MYSQL_REPLICA_CONFIG = {
    "host": os.environ.get("SQL_HOST"),
    "port": int(os.environ.get("SQL_PORT")),
    "db": os.environ.get("SQL_DB"),
    "password": os.environ.get("SQL_PASSWORD"),
    "user": os.environ.get("SQL_USER"),
    "charset": os.environ.get("SQL_CHARSET"),
    "cursorclass": pymysql.cursors.DictCursor,
}

# class to create database connection pooling
POOL = PooledDB(**configs.MYSQL_POOL_CONFIG, **configs.MYSQL_REPLICA_CONFIG)


class SqlPooled:
    """Sql connection with pooling."""

    def __init__(self):
        self._connection = POOL.connection()
        self._cursor = self._connection.cursor()

    def fetch_one(self, sql, args):
        self._cursor.execute(sql, args)
        result = self._cursor.fetchone()
        return result

    def fetch_all(self, sql, args):
        self._cursor.execute(sql, args)
        result = self._cursor.fetchall()
        return result

    def __del__(self):
        self._connection.close()
```
