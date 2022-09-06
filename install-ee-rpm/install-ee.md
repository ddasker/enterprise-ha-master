# INSTALL - MYSQL ENTERPRISE EDITION

## Introduction

Detailed Installation of MySQL Enterprise Edition 8.0 and MySQL Shell on Linux

Objective: RPM Installation of MySQL 8 Enterprise on Linux


RPM Installation of MySQL Enterprise 8 on Linux

Estimated Time: 15 minutes

### Objectives

In this lab, you will:
* Install MySQL Enterprise Edition
* Start and test MySQL Enterpriese Edition Install
* Install MySQL Shell and Connect to MySQL Enterprise 


### Prerequisites

Test code
This lab assumes you have:
* An Oracle account
* All previous labs successfully completed

* Lab standard  
    - ![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell> the command must be executed in the Operating System shell
    - ![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql> the command must be executed in a client like MySQL, MySQL Workbench
    - ![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh> the command must be executed in MySQL shell
    
## Task 1: Install MySQL Enterprise Edition using Linux RPM's

**Note:** If not already connected with SSH

- connect to **myclient** instance using Cloud Shell (**Example:** ssh -i ~/.ssh/id_rsa opc@132.145.17….)

    ```
    <copy>ssh -i ~/.ssh/id_rsa opc@<your_compute_instance_ip></copy>
    ```

    ![CONNECT](./images/06connect01-signin.png " ")


1. Install the RPM's

 **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>cd ~/workshop</copy>
    ```

 **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>sudo yum -y install *.rpm</copy>
    ```


2.	Start the MySQL daemon 

 **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>sudo systemctl start mysqld</copy>
    ```

 **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>sudo mkdir /mysql/log /mysql/temp /mysql/binlog</copy>
    ```


## Task 2: Start and test MySQL Enterprise Edition Install

1.	Start your new mysql instance

  **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>sudo /mysql/mysql-latest/bin/mysqld --defaults-file=/mysql/etc/my.cnf --user=mysqluser &</copy>
    ```

2.	Verify that process is running

  **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>ps -ef | grep mysqld</copy>
    ```

  **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>netstat -an | grep 3306</copy>
    ```


3.	Another way is searching the message “ready for connections” in error log as one of the last

  **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>grep -i ready /mysql/log/err&#95;log.log</copy>
    ```

4. Install the MySQL Shell command line utility

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
     ```
    <copy>sudo yum -y install /workshop/shell/mysql-shell-commercial-8.0.28-1.1.el8.x86_64.rpm</copy>
    ```

5.	Retrieve root password for first login:

  **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>grep -i 'temporary password' /mysql/log/err&#95;log.log</copy>
    ```

6. Login to the the mysql-enterprise installation and check the status (you will be asked to change password)

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
     ```
    <copy>mysqlsh --uri root@localhost:3306 --sql -p </copy>
    ```

7. Create New Password for MySQL Root

 **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>ALTER USER 'root'@'localhost' IDENTIFIED BY 'Welcome1!';</copy>
    ```

 **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>\status</copy>
    ```

8.	Shutdown the service

 **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>\quit</copy>
    ```


9.	Create a new administrative user called 'admin' with remote access and full privileges

 **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>mysqlsh --sql --uri root@127.0.0.1:3306 -p</copy>
    ```

 **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>CREATE USER 'admin'@'%' IDENTIFIED BY 'Welcome1!';</copy>
    ```

 **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>GRANT ALL PRIVILEGES ON *.* TO 'admin'@'%' WITH GRANT OPTION;</copy>
    ```

10.	Add the mysql bin folder to the bash profile

 **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>\quit</copy>
    ```

 **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>nano /home/opc/.bash&#95;profile</copy>
    ```

11. After the value  **# User specific environment and startup programs**. Add the following line:
    ```
    <copy>PATH=$PATH:/mysql/mysql-latest/bin:$HOME/.local/bin:$HOME/bin</copy>
    ```

12. Save the changes, log out and log in again via ssh for the changes to take effect on the user profile.  Or you can source the .bash&#95;profile file to update your environment.

 **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
   <copy>source /home/opc/.bash&#95;profile</copy>
    ```


## Learn More

* [MySQL Linux Installation](https://dev.mysql.com/doc/refman/8.0/en/binary-installation.html)
* [MySQL Shell Installation](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-install.html)

## Acknowledgements
* **Author** - Dale Dasker, MySQL Solution Engineering
* **Last Updated By/Date** - <Dale Dasker, April 2022
