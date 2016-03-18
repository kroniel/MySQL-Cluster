#Introduction

This Docker container provides a MySQL Cluster installation.

The project sources can be found on [GitHub](https://github.com/g17/MySQL-Cluster). The Docker image build is hosted at [Docker Hub](https://registry.hub.docker.com/u/h3nrik/mysql-cluster/).


#Build

    git clone https://github.com/kroniel/mcluster.git
    cd mcluster
    docker build -t kroniel/mcluster .


#Prerequisites

In order to set up the MySQL Cluster you should have prepared a proper network setup similar to the following. It includes one management node, two data nodes and two SQL nodes.

* Management Node: host name _mgm01_ (192.168.0.1).
* SQL Nodes: host name _mysqld01_ (192.168.0.100) and _mysqld02_ (192.168.0.101).
* Data Nodes: host name _ndbd01_ (192.168.0.10) and _ndbd02_ (192.168.0.11).

#Installation

Prepare a configuration file for the management node similar to the following one (but do not change the _datadir_ definitions!):

    [NDBD DEFAULT]
    NoOfReplicas=2
    DataMemory=80M
    IndexMemory=18M
    datadir=/usr/local/mysql/data

    [NDB_MGMD DEFAULT]
    datadir=/var/lib/mysql-cluster

    [NDB_MGMD]
    NodeId=1
    hostname=192.168.0.1

    [NDBD]
    NodeId=10
    hostname=192.168.0.10

    [NDBD]
    NodeId=11
    hostname=192.168.0.11

    [MYSQLD]
    NodeId=100
    hostname=192.168.0.100

    [MYSQLD]
    NodeId=101
    hostname=192.168.0.101



#Usage

1. Start the management node on host 192.168.0.1:

        docker run -d --name ndb_mgmd01 --net=host -p 192.168.0.1:1186:1186 -v /path/to/your/config.ini:/etc/mysql-cluster.ini:ro kroniel/mysql-cluster ndb_mgmd


2. Connect with the management console to the management node to issue the _SHOW_ command that displays your cluster configuration and state:

        docker run -it --rm --name ndb_mgm kroniel/mysql-cluster ndb_mgm 192.168.0.1

3. Run the data nodes on host 192.168.0.10 and 192.168.0.11:

        docker run -d --name ndbd01 --net=host kroniel/mysql-cluster ndbd 192.168.0.1
        docker run -d --name ndbd02 --net=host kroniel/mysql-cluster ndbd 192.168.0.1


4. Run the SQL nodes on host 192.168.0.100 and 192.168.0.101:

        docker run -d --name mysqld01 --net=host -p 192.168.0.100:3306:3306 kroniel/mysql-cluster mysqld 192.168.0.1
        docker run -d --name mysqld02 --net=host -p 192.168.0.101:3306:3306 kroniel/mysql-cluster mysqld 192.168.0.1

5. Now you can connect to any of the SQL nodes with your client.


#Update Nodes

You can easily update the data/SQL nodes e.g. for up-scaling the cluster:

1. Modify the _config.ini_ file mounted into the management node.
2. Stop the management node.
3. Start it again. It will do a reload of that configuration file automatically.
4. Start the new SQL/data nodes on the hosts configured in the _config.ini_ file.

#Further Ideas

In addition to this setup a MySQL proxy could be used to access the SQL nodes.
