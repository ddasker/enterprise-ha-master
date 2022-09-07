# DEPLOYING - InnoDB ReplicaSets

## Introduction

InnoDB ReplicaSets
Objective: deploying MySQL sandboxes and then creating an InnoDB ReplicaSet


*This lab walks you through creating MySQL Sandboxes, deploying InnoDB ReplicaSets, bootstrapping MySQL Router and testing failovers

Estimated Time: 15 minutes


### Objectives

In this lab, you will  do the followings:
- Connect to MySQL Shell
- Create MySQL Sandboxes
- Create InnoDB ReplicaSet 

### Prerequisites

This lab assumes you have:
* An Oracle account
* All previous labs successfully completed

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

2. Create 3 MySQL Sandboxes 

	a. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>** 

    ```
    <copy>dba.deploySandboxInstance(3310, {password: "password"})</copy>
    ```

    b. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>** 

    ```
    <copy>dba.deploySandboxInstance(3320, {password: "password"})</copy>
    ```

	c. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>dba.deploySandboxInstance(3330, {password: "password"})</copy>
    ```

## Task 2: Create ReplicaSet

1. Using the MySQL Shell Connection, connect the Shell to Sandbox on Port 3310 and create ReplicaSet

	a. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>\connect 'root@localhost:3310'</copy>
    ```

	b. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>var rs = dba.createReplicaSet("example")</copy>
    ```

	c. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>rs.status()</copy>
    ```

2. Add 2 instances to ReplicaSet

    a. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>rs.addInstance('root@localhost:3320')</copy>
    ```

	b. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>rs.addInstance('root@localhost:3330')</copy>
    ```

	c. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>rs.status()</copy>
    ```

## Task 3: Test failovers

1. Test changing the Primary.  This is good for instances where you want to safely failover to a new Replica

	a. Failover to 3320 instance
    
    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>rs.setPrimaryInstance('root@localhost:3320')</copy>
    ```

	b. Check status

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>rs.status()</copy>
    ```

	c. Failover back to 3310 instance
    
    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>rs.setPrimaryInstance('root@localhost:3310')</copy>
    ```

	d. Check status (**Note** You can see extended details by passing the {extended: [1|2} })

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>rs.status()</copy>
    ```


## Task 4: Deploy MySQL Router

1.	Close MySQL Shell and create directory for MySQL Router configuration and data

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>\quit</copy>
    ```

	**![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 

    ```
    <copy>mkdir ~/mysqlrouter</copy>
    ```

	**![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 

    ```
    <copy>cd ~/mysqlrouter</copy>
    ```

2.	Bootstrap MySQL Router and Deploy Router against 3310 Instance (Which is now the Source) 

	**![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 

    ```
    <copy>mysqlrouter --bootstrap root@localhost:3310 -d /home/opc/mysqlrouter</copy>
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

3.	Using the administrative connection revoke all privileges using and administrative connection and verify

	**![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**
	```
    <copy>REVOKE ALL PRIVILEGES ON *.* FROM 'appuser1'@'127.0.0.1';</copy>
    ```
	**![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**
	```
    <copy>SHOW GRANTS FOR 'appuser1'@'127.0.0.1';</copy>
    ```   
4.	Close and reopen appuser session, do you see schemas?

## Task 5: Restore user privileges
1.	Using the administrative connection restore user privileges to reuse it in next labs

	**![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**
    ```
    <copy>GRANT ALL PRIVILEGES ON employees.* TO 'appuser1'@'127.0.0.1';</copy>
    ```




## Learn More

* [CREATE USER](https://dev.mysql.com/doc/refman/8.0/en/create-user.html)
* [MySQL Access Control Lists](https://dev.mysql.com/doc/refman/8.0/en/access-control.html)

## Acknowledgements
* **Author** - Dale Dasker, MySQL Solution Engineering

