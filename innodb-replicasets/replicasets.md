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

	a. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 

    ```
    <copy>dba.deploySandboxInstance(3310, {password: "password"})</copy>
    ```

	b. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 

    ```
    <copy>dba.deploySandboxInstance(3320, {password: "password"})</copy>
    ```

	c. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 

    ```
    <copy>dba.deploySandboxInstance(3330, {password: "password"})</copy>
    ```

## Task 2: Create ReplicaSet

1. Using the MySQL Shell Connection, connect the Shell to Sandbox on Port 3310 and create ReplicaSet

	a. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 

    ```
    <copy>\connect 'root@localhost:3310'</copy>
    ```

	b. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 

    ```
    <copy>var rs = dba.createReplicaSet("example")</copy>
    ```

	c. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 

    ```
    <copy>rs.status()</copy>
    ```

2. Add 2 instances to ReplicaSet

    a. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 

    ```
    <copy>rs.addInstance('root@localhost:3320')</copy>
    ```

	b. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 

    ```
    <copy>rs.addInstance('root@localhost:3330')</copy>
    ```

	c. **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 

    ```
    <copy>rs.status()</copy>
    ```

## Task 3: Test failovers

1. Test changing the Primary.  This is good for instances where you want to safely failover to a new Replica

	a. Failover to 3320 instances
    
    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 

    ```
    <copy>rs.setPrimaryInstance('root@localhost:3320')</copy>
    ```

	b. Check status

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 

    ```
    <copy>rs.status()</copy>
    ```

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 

    ```
    <copy>SELECT * FROM employees;</copy>
    ```

2. Switch to the administrative connection revoke privilege on city to appuser

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 

    ```
    <copy>REVOKE SELECT ON employees.* FROM 'appuser1'@'127.0.0.1';</copy>
    ```

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 

    ```
    <copy>SHOW GRANTS FOR 'appuser1'@'127.0.0.1';</copy>
    ```
3. Repeat the select on appuser connection for the user. There is a difference?

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 

    ```
    <copy>SELECT * FROM employees;</copy>
    ```

## Task 4: Use appuser1 connection
1.	Close and reopen the appuser1 connection for the user, then repeat above commands. There is a difference? 

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 

    ```
    <copy>exit</copy>
    ```

	**![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 

    ```
    <copy>mysql -u appuser1 -p -h 127.0.0.1 -P 3306</copy>
    ```
	**![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 

    ```
    <copy>USE employees;</copy>
    ```
    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>SELECT * FROM employees;</copy>
    ```
2.	Switch to the administrative connection revoke ‘USAGE’ privilege using and administrative connection and verify (tip: this privilege can’t be revoked…)

	**![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**
	```
	<copy>REVOKE USAGE ON *.* FROM 'appuser1'@'127.0.0.1';</copy>
	```
	**![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**
	```
	<copy>SHOW GRANTS FOR 'appuser1'@'127.0.0.1';</copy>
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
