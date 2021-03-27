Abdirizak kulmiye
CNA350
				        							Real World Project: Database Shard GitHub.
This project is how to architecture backend servers and shard the Data with two servers using Maxscale server and show the client he is accessing one server but technically he is accessing two master MariaDB servers connecting with maxscale.

Pre-requisites.
The prerequisites for this project is you need to have virtual machine running with OS ubuntu 20.04, and you need to install docker and docker-compose.

this is how to install docker community edition.

sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
sudo apt update
sudo apt install docker-ce

Check the Docker status with command.
sudo systemctl run docker
sudo systenctl enable docker
sudo systemctl status docker.

Installation Docker Compose:
sudo apt install docker-compose.

install client access to MariaDB 
sudo apt install MariaDB-client.
Set up and clone maxscale-docker repository from zohan GitHub.
git clone https://github.com/Zohan/maxscale-docker.git
to run the docker-compose up you must be in right directory "maxscale-docker/maxscale"
docker-compose up -d
kumiye73@linux: ~/docker/maxscale-docker/maxscale$ docker-compose up -d
Starting maxscale_master_1 ... done
Starting maxscale_master2_1 ... done
Starting maxscale_maxscale_1 ... done
kumiye73@linux: ~/docker/maxscale-docker/maxscale$
check all servers states and use this command.
kumiye73@linux:~/docker/maxscale-docker/maxscale$ docker ps -a
CONTAINER ID        IMAGE                     COMMAND                  CREATED             STATUS                         PORTS                    NAMES
950c32a177b6        MariaDB/maxscale: latest   "/usr/bin/tini -- do…"   10 days ago         Exited (0) About an hour ago                            maxscale_maxscale_1
6104009270cd        MariaDB: latest            "docker-entrypoint.s…"   10 days ago         Up About an hour               0.0.0.0:4001->3306/tcp   maxscale_master_1
401172fea7d8        MariaDB: latest            "docker-entrypoint. s…"   10 days ago         Up 52 seconds                  0.0.0.0:4003->3306/tcp   maxscale_master2_1
Check if the servers are running and connecting the ports, use this command.
docker-compose exec maxscale maxctrl list servers.
┌────────────────┬─────────┬──────┬─────────────┬─────────────────┬───────────┐
│ Server         │ Address │ Port │ Connections │ State           │ GTID      │
├────────────────┼─────────┼──────┼─────────────┼─────────────────┼───────────┤
│ zip_master_one │ master  │ 3306 │ 0           │ Master, Running │ 0-3000-32 │
├────────────────┼─────────┼──────┼─────────────┼─────────────────┼───────────┤
│ zip_master_two │ master2 │ 3306 │ 0           │ Running         │ 0-3000-31 │
└────────────────┴─────────┴──────┴─────────────┴─────────────────┴───────────┘
check if master down how to master2 become master1 and use this command. 
docker-compose stop master.
 docker-compose stop master.
Stopping maxscale_master_1 ... done
 docker-compose exec maxscale maxctrl list servers.
┌────────────────┬─────────┬──────┬─────────────┬─────────────────┬───────────┐
│ Server         │ Address │ Port │ Connections │ State           │ GTID      │
├────────────────┼─────────┼──────┼─────────────┼─────────────────┼───────────┤
│ zip_master_one │ master  │3306  │ 0           │ Down            │ 0-3000-32 │
├────────────────┼─────────┼──────┼─────────────┼─────────────────┼───────────┤
│ zip_master_two │ master2 │ 3306 │ 0           │ Master, Running │ 0-3000-31 │
└────────────────┴─────────┴──────┴─────────────┴─────────────────┴───────────┘

now create client to access the databases.using this commands
this is the way you can access your data from mysql console.

mysql -umaxuser -pmaxpwd -h 127.0.0.1 -P 4000
Welcome to the MariaDB monitor.  Commands end with; or \g.
Your MariaDB connection id is 4
Server version: 10.5.9-MariaDB-1:10.5.9+maria~focal-log mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 

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
Running python
This is the output  when I run python zipcodes.py


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


The first 10 rows of zipcodes_two are:
(42040, 'STANDARD', 'FARMINGTON', 'KY', 'PRIMARY', '36.67', '-88.53', 'NA-US-KY-FARMINGTON', 'FALSE', '465', '896', '11562973')
(41524, 'STANDARD', 'FEDSCREEK', 'KY', 'PRIMARY', '37.4', '-82.24', 'NA-US-KY-FEDSCREEK', 'FALSE', '', '', '')
(42533, 'STANDARD', 'FERGUSON', 'KY', 'PRIMARY', '37.06', '-84.59', 'NA-US-KY-FERGUSON', 'FALSE', '429', '761', '9555412')
(40022, 'STANDARD', 'FINCHVILLE', 'KY', 'PRIMARY', '38.15', '-85.31', 'NA-US-KY-FINCHVILLE', 'FALSE', '437', '839', '19909942')
(40023, 'STANDARD', 'FISHERVILLE', 'KY', 'PRIMARY', '38.16', '-85.42', 'NA-US-KY-FISHERVILLE', 'FALSE', '1884', '3733', '113020684')
(41743, 'PO BOX', 'FISTY', 'KY', 'PRIMARY', '37.33', '-83.1', 'NA-US-KY-FISTY', 'FALSE', '', '', '')
(41219, 'STANDARD', 'FLATGAP', 'KY', 'PRIMARY', '37.93', '-82.88', 'NA-US-KY-FLATGAP', 'FALSE', '708', '1397', '20395667')
(40935, 'STANDARD', 'FLAT LICK', 'KY', 'PRIMARY', '36.82', '-83.76', 'NA-US-KY-FLAT LICK', 'FALSE', '752', '1477', '14267237')
(40997, 'STANDARD', 'WALKER', 'KY', 'PRIMARY', '36.88', '-83.71', 'NA-US-KY-WALKER', 'FALSE', '', '', '')
(41139, 'STANDARD', 'FLATWOODS', 'KY', 'PRIMARY', '38.51', '-82.72', 'NA-US-KY-FLATWOODS', 'FALSE', '3692', '6748', '121902277')


The largest zip code number in zipcodes_one is:
(47750,)
The smallest zip code number in zipcodes_two is:
(38257,)
## I worked this project with my class mate Vlado Luma Saied Mohamed and Igor.
Sources
https://docs.docker.com/compose/install/
https://mariadb.com/kb/en/mariadb-maxscale-25-simple-sharding-with-two-servers/
https://docs.docker.com/compose/compose-file/
https://github.com/Zohan/maxscale-docker
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04
https://docker-curriculum.com/#what-are-containers-

