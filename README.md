VCC Database
===

## Introduction

This repository is a fork from the [original](https://github.com/announce/vcc-base) to view vulnerability-contributing commits (VCC) data via [PostgreSQL](https://www.postgresql.org/). This fork gives instructions on how to extract the database dump into PostgreSQL using [command line tools](https://postgresapp.com/documentation/cli-tools.html) from the [Postgres.app](https://postgresapp.com/) in the Apple macOS environment. All credit to the database goes to the original authors listed in the related paper below.

#### Related Paper
* *Henning Perl, Sergej Dechand, Matthew Smith, Daniel Arp, Fabian Yamaguchi, Konrad Rieck, Sascha Fahl, and Yasemin Acar. 2015. VCCFinder: Finding Potential Vulnerabilities in Open-Source Projects to Assist Code Audits. In Proceedings of the 22nd ACM SIGSAC Conference on Computer and Communications Security (CCS ’15). Association for Computing Machinery, New York, NY, USA, 426–437. DOI:https://doi.org/10.1145/2810103.2813604*

## Requirements

#### Current Environment and Tools
* macOS
* [Postgres.app](https://postgresapp.com/)

## Download and Extract Data

1. Download [vcc-database.dump.bz2](https://www.dropbox.com/s/qtap9m1k33879r7/vcc-database.dump.bz2?dl=0) (417.2 MB)
2. Extract `vcc-database.dump` using the following command via Terminal in the directory where the file was downloaded. Export the file to the current directory or adjust path to desired directory. (2.49 GB)

```
$ bzip2 -dc vcc-database.dump.bz2 > vcc-database.dump
```

## PostgreSQL Installation

1. Download and install [Postgres.app](https://postgresapp.com/). A default server is available and can be started as clicking `Initialize Server`. Default databases are provided as well. Double-click a database to start `psql`, a terminal-based front-end to PostgreSQL.

    - Postgres.app includes many [command line tools](https://postgresapp.com/documentation/cli-tools.html). If you want to use them, you must configure the $PATH variable. **The remainder of this document uses commands via the command line tools in Terminal.**
    ```
    $ sudo mkdir -p /etc/paths.d && echo /Applications/Postgres.app/Contents/Versions/latest/bin | sudo tee /etc/paths.d/postgresapp
    ```

    - Once you enter the command you may need to enter your password since `sudo` is being used. After the command executes, you should receive output such as:
    ```
    $ /Applications/Postgres.app/Contents/Versions/latest/bin
    ```

    - Close the Terminal and type:
    ```
    $ which psql
    ```

    - If Postgres.app was successfully added the $PATH, then you should receive output such as:
    ```
    $ /Applications/Postgres.app/Contents/Versions/latest/bin/psql
    ```

2. Type `psql` to initiate PostgreSQL via Terminal. You should receive output such as this, which confirms you have initialized PostgreSQL via the Terminal:
    ```
    $ psql
    psql (12.3)
    Type "help" for help.

    myUser=#
    ```

3. Create a new database for VCC data, named `vcc`. If successful, should receive output of `CREATE DATABASE`:
    ```
    myUser=# create database vcc;
    CREATE DATABASE
    ```
    - To make sure, you can run the following command to list databases:
    ```
    myUser=# \l
    ```
    - Connect to the new database:
    ```
    myUser=# \c vcc
    You are now connected to database "vcc" as user "myUser".
    ```
    - Create extension `hstore`. The `hstore` module implements the hstore data type for storing key-value pairs in a single value. If successful, should receive output of `CREATE EXTENSION`:
    ```
    vcc=# create extension hstore;
    CREATE EXTENSION
    ```
4. Import `vcc-database.dump` in to `vcc` database
    - Navigate to local directory where `vcc-database.dump` is located
    - Run the following command to transfer the `vcc-database.dump` file into our `vcc` database:
    ```
    psql -U myUser -p 5432 -h localhost -d vcc < vcc-database.dump -v ON_ERROR_STOP=1;
    ```

5. List schemas in `vcc` database:
    ```
    vcc=# \dn
      List of schemas
      Name  |  Owner
    --------+----------
     export | myUser
     public | postgres
    (2 rows)
    ```

6. List tables of `export` schema in `vcc` database:
    ```
    vcc=# \dt export.
                List of relations
     Schema |     Name     | Type  |  Owner
    --------+--------------+-------+---------
     export | commits      | table | myUser
     export | cves         | table | myUser
     export | repositories | table | myUser
    (3 rows)
    ```

7. Example query:
    - Query count of records in `commits` table. If successful, should receive result:
    ```
    vcc=# SELECT COUNT(*) FROM export.commits;
     count
    --------
     351409
    (1 row)
    ```

8. Example query to file:
    - Copy `10` records from `export.commits` to file (`results.csv`) on local machine in directory where `psql` command was initiated. If successful, should see result of `COPY {number of records}` after command:
    ```
    vcc=# \COPY (SELECT * FROM export.commits LIMIT 10) TO 'results.csv' CSV HEADER;
    COPY 10
    ```


- Other `psql` commands
    - To view current users, type:
    ```
    myUser=# \du
    ```
    - Which should provide an output such as:
    ```
                                        List of roles
    Role name   |                         Attributes                         | Member of
    ------------+------------------------------------------------------------+-----------
    myUser      | Superuser, Create role, Create DB                          | {}
    postgres    | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
    ```

    - By default there should be a user named `postgres` and another user for your current account name, e.g. If your computer account name is `myUser` then your other PostgreSQL name would be `myUser`.

    - To view current databases, type:
    ```
    myUser=# \l
    ```
    - Which should provide an output such as:
    ```
                                         List of databases
    Name          |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
    --------------+----------+----------+-------------+-------------+-----------------------
    myUser        | myUser   | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
    postgres      | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
    template0     | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
                  |          |          |             |             | postgres=CTc/postgres
    template1     | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
                  |          |          |             |             | postgres=CTc/postgres
    vcc           | myUser   | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
    (6 rows)
    ```

    - Command to drop a database. If successful, should receive output `DROP DATABASE`:
    ```
    myUser=# drop database vcc;
    DROP DATABASE
    ```
