# DEPLOYING - InnoDB ClusterSets

## Introduction

InnoDB ClusterSets:
Objective: deploying MySQL sandboxes and then creating an InnoDB ClusterSets


*This lab walks you through creating MySQL Sandboxes, deploying InnoDB ClusterSets, bootstrapping MySQL Router and testing failovers

Estimated Time: 15 minutes


### Objectives

In this lab, you will  do the followings:
- Connect to MySQL Shell
- Create MySQL Sandboxes
- Create InnoDB ClusterSets 

### Prerequisites

This lab assumes you have:
* An Oracle account
* All previous labs successfully completed

### MySQL Instances ports
* "Portland": 3310, 3320, 3330
* "Seattle": 3410, 3420, 3430

* Lab standard  
    - ![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell> the command must be executed in the Operating System shell
    - ![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql> the command must be executed in a client like MySQL, MySQL Workbench
    - ![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh> the command must be executed in MySQL shell
    
**Notes:**
- Open a notepad file and  your linux Private IP on student###-serverA 

- serverA  PRIVATE ip: (client_ip)

## Task 1: Connect to mysql-enterprise on Server

1. Connect to your MySQL Shell

   **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 

    ```
    <copy>mysqlsh</copy>
    ```

2. Create 3 Additional MySQL Sandboxes 

	a. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>** 

    ```
    <copy>dba.deploySandboxInstance(3410, {password: "password"})</copy>
    ```

    b. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>** 

    ```
    <copy>dba.deploySandboxInstance(3420, {password: "password"})</copy>
    ```

	c. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>dba.deploySandboxInstance(3430, {password: "password"})</copy>
    ```

## Task 2: Create an InnoDB ClusterSet 

1. Using the MySQL Shell Connection, connect the Shell to Sandbox on Port 3310 and create ClusterSet starting with 3410 Instance

	a. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>\connect root@localhost:3310</copy>
    ```

	b. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>var PortlandCluster = dba.getCluster()</copy>
    ```

	c. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>PortlandCluster.status()</copy>
    ```

	d. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>NWClusterSet = PortlandCluster.createClusterSet("PortlandSet")</copy>
    ```

	e. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>NWClusterSet.status()</copy>
    ```

	f. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>SeattleCluster = NWClusterSet.createReplicaCluster("127.0.0.1:3410","SeattleCluster")</copy>
    ```

	g. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>SeattleCluster.status()</copy>
    ```

	h. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>NWClusterSet.status()</copy>
    ```

2. Add 2 instances to Secondary (Replica) InnoDB Cluster

    a. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>SeattleCluster.addInstance('root@localhost:3420')</copy>
    ```

	b. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>SeattleCluster.addInstance('root@localhost:3430')</copy>
    ```

	c. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>SeattleCluster.status()</copy>
    ```

	d. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>\connect root@localhost:3410</copy>
    ```

	e. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>\sql</copy>
    ```

	f. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>SHOW DATABASES;</copy>
    ```

	g. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>USE world;</copy>
    ```

	h. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>SHOW TABLES;</copy>
    ```

	i. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>\js</copy>
    ```

	j. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>\connect root@localhost:3310</copy>
    ```

## Task 3: Test failovers

1. Test changing the Primary.  This is good for instances where you want to safely failover to a new Replica

	a. Failover to 3320 instance
    
    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>PortlandCluster.setPrimaryInstance("root@localhost:3320")</copy>
    ```

	b. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>\connect root@localhost:3320</copy>
    ```

	c. Check status

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>PortlandCluster.status()</copy>
    ```

	d. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>NWClusterSet = dba.getClusterSet()</copy>
    ```

	e. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>NWClusterSet.status()</copy>
    ```

	f. Failover back to 3310 instance
    
    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>PortlandCluster.setPrimaryInstance("root@localhost:3310")</copy>
    ```

	g. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>\connect root@localhost:3310</copy>
    ```

	h. Check status (**Note** You can see extended details by passing the {extended: [1|2} })

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>PortlandCluster.status()</copy>
    ```

	i. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>NWClusterSet.status()</copy>
    ```


## Task 4: Deploy MySQL Router

1.	Create a new SSH Shell window to your Compute Instance and create a directory for MySQL Router configuration and data

	**![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 

    ```
    <copy>cd ~/mysqlrouter</copy>
    ```

2.	Bootstrap MySQL Router and Deploy Router against 3310 Instance (Which is now the Source) 

	**![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 

    ```
    <copy>mysqlrouter --bootstrap root@localhost:3310 -d /home/opc/mysqlrouter --name='Portland'</copy>
    ```

	**![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 

    ```
    <copy>./start.sh &</copy>
    ```

	**![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 

    ```
    <copy>ps -ef | grep mysqlrouter</copy>
    ```

	**![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 

    ```
    <copy>mysql -P6446 --protocol=tcp -uroot -ppassword</copy>
    ```

	**![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**
	```
    <copy>SELECT @@port;</copy>
    ```

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>NWClusterSet.listRouters()</copy>
    ```

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>NWClusterSet.routingOptions()</copy>
    ```

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>NWClusterSet.describe()</copy>
    ```

3.	Failover the Source and check if the Router follows

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
	```
    <copy>PortlandCluster.setPrimaryInstance('root@localhost:3320')</copy>
    ```

	**![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**

	```
    <copy>SELECT @@port;</copy>
    ```   

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
	```
    <copy>PortlandCluster.setPrimaryInstance('root@localhost:3310')</copy>
    ```

4.	Failover to the Replica Cluster

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
	```
    <copy>NWClusterSet.setPrimaryCluster('SeattleCluster')</copy>
    ```

	**![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**
	```
    <copy>SELECT @@port;</copy>
    ```   

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
	```
    <copy>\connect root@localhost:3410</copy>
    ```   

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
	```
    <copy>NWClusterSet.status()</copy>
    ```   

	**![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**
	```
    <copy>SELECT @@port;</copy>
    ```   

5.  Still working out what to do next

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
	```
    <copy>dba.startSandboxInstance(3320)</copy>
    ```

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>PortlandCluster.status()</copy>
    ```



## Learn More

* [CREATE USER](https://dev.mysql.com/doc/refman/8.0/en/create-user.html)
* [MySQL Access Control Lists](https://dev.mysql.com/doc/refman/8.0/en/access-control.html)

## Acknowledgements
* **Author** - Dale Dasker, MySQL Solution Engineering

