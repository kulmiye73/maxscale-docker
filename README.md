Abdirizak kulmiye
CNA350
		 Real World Project: Database Shard GitHub.

this project is how to architecture  backend servers and shard the Data with two servers using Maxscale server,and show the client he is accessing one server but technically he is accessing two master MariaDB servers connecting with maxscale.

Pre-requisites.

The prerequisites for this project is you need to have virtual machine running with OS ubuntu 20.04,and you need to install docker and docker-compose.
this is how to install docker community edition.

sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
sudo apt update
sudo apt install docker-ce.

To check if the docker installed use these commands.
sudo systemctl run docker
sudo systenctl enable docker
sudo systemctl status docker.


 install Docker Compose:

sudo apt install docker-compose.

install client access to mariadb 

sudo apt install mariadb-client.
clone maxscale-docker repository from zohan github.
https://github.com/Zohan/maxscale-docker.git
to run the docker-compose up you must be in right directory "maxscale-docker/maxscale"

docker-compose up -d

Starting maxscale_master_1  ... done
Starting maxscale_master2_1 ... done
Starting maxscale_maxscale_1 ... done
To check all containers are up use this command

sudo docker ps -a

CONTAINER ID        IMAGE                     COMMAND                  CREATED             STATUS              PORTS                                                      NAMES
950c32a177b6        mariadb/maxscale:latest   "/usr/bin/tini -- do…"   8 minutes ago       Up 8 minutes        0.0.0.0:4000->4000/tcp, 3306/tcp, 0.0.0.0:8989->8989/tcp   maxscale_maxscale_1
6104009270cd        mariadb:latest            "docker-entrypoint.s…"   8 minutes ago       Up About a minute   0.0.0.0:4001->3306/tcp                                     maxscale_master_1
401172fea7d8        mariadb:latest            "docker-entrypoint.s…"   8 minutes ago       Up 8 minutes        0.0.0.0:4003->3306/tcp                                     maxscale_master2_1


check if all servers states and use this command.

docker-compose exec maxscale maxctrl list servers
┌────────────────┬─────────┬──────┬─────────────┬─────────────────┬───────────┐
│ Server         │ Address │ Port │ Connections │ State           │ GTID      │
├────────────────┼─────────┼──────┼─────────────┼─────────────────┼───────────┤
│ zip_master_one │ master  │ 3306 │ 0           │ Running         │           │
├────────────────┼─────────┼──────┼─────────────┼─────────────────┼───────────┤
│ zip_master_two │ master2 │ 3306 │ 0           │ Master, Running │ 0-3000-31 │
└────────────────┴─────────┴──────┴─────────────┴─────────────────┴───────────


check if master down how to master2 become master1 and use this command 
docker-compose stop master.
 docker-compose stop master
Stopping maxscale_master_1 ... done
 docker-compose exec maxscale maxctrl list servers
┌────────────────┬─────────┬──────┬─────────────┬─────────────────┬───────────┐
│ Server         │ Address │ Port │ Connections │ State           │ GTID      │
├────────────────┼─────────┼──────┼─────────────┼─────────────────┼───────────┤
│ zip_master_one │ master  │ 3306 │ 0           │ Down            │           │
├────────────────┼─────────┼──────┼─────────────┼─────────────────┼───────────┤
│ zip_master_two │ master2 │ 3306 │ 0           │ Master, Running │ 0-3000-31 


now create client to access the databases.using this commands
this is the way you can access your data from mysql console;
mysql -umaxuser -pmaxpwd -h 127.0.0.1 -P 4000

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 3
Server version: 10.5.9-MariaDB-1:10.5.9+maria~focal-log mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| zipcodes_one       |
| zipcodes_two       |
+--------------------+
5 rows in set (0.001 sec)

The last 10 rows of zipcodes_one


 MariaDB [(none)]> SELECT *FROM zipcodes_one.zipcodes_one LIMIT 9990,10;
+---------+-------------+----------------+-------+--------------+-----------+------------+-------------------------+---------------+-----------------+---------------------+------------+
| Zipcode | ZipCodeType | City           | State | LocationType | Coord_Lat | Coord_Long | Location                | Decommisioned | TaxReturnsFiled | EstimatedPopulation | TotalWages |
+---------+-------------+----------------+-------+--------------+-----------+------------+-------------------------+---------------+-----------------+---------------------+------------+
|   40843 | STANDARD    | HOLMES MILL    | KY    | PRIMARY      | 36.86     | -83        | NA-US-KY-HOLMES MILL    | FALSE         |                 |                     |            |
|   41425 | STANDARD    | EZEL           | KY    | PRIMARY      | 37.89     | -83.44     | NA-US-KY-EZEL           | FALSE         | 390             | 801                 | 10204009   |
|   40118 | STANDARD    | FAIRDALE       | KY    | PRIMARY      | 38.11     | -85.75     | NA-US-KY-FAIRDALE       | FALSE         | 4398            | 7635                | 122449930  |
|   40020 | PO BOX      | FAIRFIELD      | KY    | PRIMARY      | 37.93     | -85.38     | NA-US-KY-FAIRFIELD      | FALSE         |                 |                     |            |
|   42221 | PO BOX      | FAIRVIEW       | KY    | PRIMARY      | 36.84     | -87.31     | NA-US-KY-FAIRVIEW       | FALSE         |                 |                     |            |
|   41426 | PO BOX      | FALCON         | KY    | PRIMARY      | 37.78     | -83        | NA-US-KY-FALCON         | FALSE         |                 |                     |            |
|   40932 | PO BOX      | FALL ROCK      | KY    | PRIMARY      | 37.22     | -83.78     | NA-US-KY-FALL ROCK      | FALSE         |                 |                     |            |
|   40119 | STANDARD    | FALLS OF ROUGH | KY    | PRIMARY      | 37.6      | -86.55     | NA-US-KY-FALLS OF ROUGH | FALSE         | 760             | 1468                | 20771670   |
|   42039 | STANDARD    | FANCY FARM     | KY    | PRIMARY      | 36.75     | -88.79     | NA-US-KY-FANCY FARM     | FALSE         | 696             | 1317                | 20643485   |
|   40319 | PO BOX      | FARMERS        | KY    | PRIMARY      | 38.14     | -83.54     | NA-US-KY-FARMERS        | FALSE         |                 |                     |            |
+---------+-------------+----------------+-------+--------------+-----------+------------+-------------------------+---------------+-----------------+---------------------+------------+
10 rows in set, 3 warnings (0.084 sec)

The first 10 rows of zipcodes_two 

MariaDB [(none)]> SELECT *FROM zipcodes_two.zipcodes_two LIMIT 10;
+---------+-------------+-------------+-------+--------------+-----------+------------+----------------------+---------------+-----------------+---------------------+------------+
| Zipcode | ZipCodeType | City        | State | LocationType | Coord_Lat | Coord_Long | Location             | Decommisioned | TaxReturnsFiled | EstimatedPopulation | TotalWages |
+---------+-------------+-------------+-------+--------------+-----------+------------+----------------------+---------------+-----------------+---------------------+------------+
|   42040 | STANDARD    | FARMINGTON  | KY    | PRIMARY      | 36.67     | -88.53     | NA-US-KY-FARMINGTON  | FALSE         | 465             | 896                 | 11562973   |
|   41524 | STANDARD    | FEDSCREEK   | KY    | PRIMARY      | 37.4      | -82.24     | NA-US-KY-FEDSCREEK   | FALSE         |                 |                     |            |
|   42533 | STANDARD    | FERGUSON    | KY    | PRIMARY      | 37.06     | -84.59     | NA-US-KY-FERGUSON    | FALSE         | 429             | 761                 | 9555412    |
|   40022 | STANDARD    | FINCHVILLE  | KY    | PRIMARY      | 38.15     | -85.31     | NA-US-KY-FINCHVILLE  | FALSE         | 437             | 839                 | 19909942   |
|   40023 | STANDARD    | FISHERVILLE | KY    | PRIMARY      | 38.16     | -85.42     | NA-US-KY-FISHERVILLE | FALSE         | 1884            | 3733                | 113020684  |
|   41743 | PO BOX      | FISTY       | KY    | PRIMARY      | 37.33     | -83.1      | NA-US-KY-FISTY       | FALSE         |                 |                     |            |
|   41219 | STANDARD    | FLATGAP     | KY    | PRIMARY      | 37.93     | -82.88     | NA-US-KY-FLATGAP     | FALSE         | 708             | 1397                | 20395667   |
|   40935 | STANDARD    | FLAT LICK   | KY    | PRIMARY      | 36.82     | -83.76     | NA-US-KY-FLAT LICK   | FALSE         | 752             | 1477                | 14267237   |
|   40997 | STANDARD    | WALKER      | KY    | PRIMARY      | 36.88     | -83.71     | NA-US-KY-WALKER      | FALSE         |                 |                     |            |
|   41139 | STANDARD    | FLATWOODS   | KY    | PRIMARY      | 38.51     | -82.72     | NA-US-KY-FLATWOODS   | FALSE         | 3692            | 6748                | 121902277  |
+---------+-------------+-------------+-------+--------------+-----------+------------+----------------------+---------------+-----------------+---------------------+------------+
10 rows in set (0.018 sec)

MariaDB [(none)]> 

There is another way you can access the zipcodes-one and zipcodes_two database from using python, and this is the code.


import pymysql

db = pymysql.connect(host="10.0.0.235", port=4000, user="maxuser", passwd="maxpwd")
cursor = db.cursor()

print('The last 10 rows of zipcodes_one are:')
cursor.execute("SELECT * FROM zipcodes_one.zipcodes_one LIMIT 9990,10;")
results = cursor.fetchall()
for result in results:
    print(result)

F:\flask.py\venv\Scripts\python.exe F:/CNA330/CNA_3301/pandas/Shard-python.py
The last 10 rows of zipcodes_one are:
(40843, 'STANDARD', 'HOLMES MILL', 'KY', 'PRIMARY', '36.86', '-83', 'NA-US-KY-HOLMES MILL', 'FALSE', '', '', '')
(41425, 'STANDARD', 'EZEL', 'KY', 'PRIMARY', '37.89', '-83.44', 'NA-US-KY-EZEL', 'FALSE', '390', '801', '10204009')
(40118, 'STANDARD', 'FAIRDALE', 'KY', 'PRIMARY', '38.11', '-85.75', 'NA-US-KY-FAIRDALE', 'FALSE', '4398', '7635', '122449930')
(40020, 'PO BOX', 'FAIRFIELD', 'KY', 'PRIMARY', '37.93', '-85.38', 'NA-US-KY-FAIRFIELD', 'FALSE', '', '', '')
(42221, 'PO BOX', 'FAIRVIEW', 'KY', 'PRIMARY', '36.84', '-87.31', 'NA-US-KY-FAIRVIEW', 'FALSE', '', '', '')
(41426, 'PO BOX', 'FALCON', 'KY', 'PRIMARY', '37.78', '-83', 'NA-US-KY-FALCON', 'FALSE', '', '', '')
(40932, 'PO BOX', 'FALL ROCK', 'KY', 'PRIMARY', '37.22', '-83.78', 'NA-US-KY-FALL ROCK', 'FALSE', '', '', '')
(40119, 'STANDARD', 'FALLS OF ROUGH', 'KY', 'PRIMARY', '37.6', '-86.55', 'NA-US-KY-FALLS OF ROUGH', 'FALSE', '760', '1468', '20771670')
(42039, 'STANDARD', 'FANCY FARM', 'KY', 'PRIMARY', '36.75', '-88.79', 'NA-US-KY-FANCY FARM', 'FALSE', '696', '1317', '20643485')
(40319, 'PO BOX', 'FARMERS', 'KY', 'PRIMARY', '38.14', '-83.54', 'NA-US-KY-FARMERS', 'FALSE', '', '', '')


mport pymysql

db = pymysql.connect(host="10.0.0.235", port=4000, user="maxuser", passwd="maxpwd")
cursor = db.cursor()

print('The largest zipcode number in zipcodes_one is:')


cursor = db.cursor()
cursor.execute("SELECT Zipcode FROM zipcodes_one.zipcodes_one ORDER BY Zipcode DESC LIMIT 1;")
results = cursor.fetchall()
for result in results:
    print(result)

:\flask.py\venv\Scripts\python.exe F:/CNA330/CNA_3301/pandas/Shard-python.py
The largest zipcode number in zipcodes_one is:
(47750,)

Process finished with exit code 0

import pymysql

db = pymysql.connect(host="10.0.0.235", port=4000, user="maxuser", passwd="maxpwd")
cursor = db.cursor()

print('The smallest zipcode number in zipcodes_two is:')


cursor = db.cursor()
cursor.execute("SELECT Zipcode FROM zipcodes_two.zipcodes_two ORDER BY Zipcode ASC LIMIT 1")
results = cursor.fetchall()
for result in results:
    print(result)

F:\flask.py\venv\Scripts\python.exe F:/CNA330/CNA_3301/pandas/Shard-python.py
The smallest zipcode number in zipcodes_two is:
(38257,)

## I worked this project with my class mate Vlado Luma Saied Mohamed and Igor and I used some resource from github and google.

https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04
https://docker-curriculum.com/#docker-images
https://github.com/mariadb-corporation/MaxScale/tree/2.3/Documentation
