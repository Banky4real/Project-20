## **Documentation for Project 20**
## **Migration to the Сloud with containerization. Part 1 - Docker & Docker Compose**

### Assembling our application from the Database Layer by making use of a prebuilt Mysql Database Container available on docker hub registry and configuring it to receive requests from our php application.

`docker pull mysql/mysql-server:latest`

![Pulling-mysql-server-image-from-docker-hub](./Images/Pulling-mysql-server-image-from-docker-hub.png)

![mysql-server-image-present-in-local-repo](./Images/mysql-server-image-present-in-local-repo.png)

### Deploying the MySQL Container to my Docker Engine by running the mysql image as a container

`docker run --name db_mysql -e MYSQL_ROOT_PASSWORD=admin12345 -d mysql/mysql-server:latest`

`docker ps -a`

![deploying-mysql-container-to-my-docker-engine](./Images/deploying-mysql-container-to-my-docker-engine.png)

### Connecting to the mysql Docker Container in two different ways: METHOD 1. By connecting directly to the container running the mysql server

`docker exec -it db_mysql mysql -uroot -p`

![Method-1-Connecting-directly-to-the-container-running-my-sql-server](./Images/Method-1-Connecting-directly-to-the-container-running-my-sql-server.png)

### Changing the mysql server root password to protect our database

`ALTER USER 'root'@'localhost' IDENTIFIED BY 'banky12345';`

`FLUSH PRIVILEGES;`

![mysql-server-root-password-changed-to-protect-database](./Images/mysql-server-root-password-changed-to-protect-database.png)

### Connecting to the mysql Docker Container in two different ways: METHOD 2. By connecting to the mysql server container on a bridge network

### Creating a Bridge Network where our container will run our application

`docker network create --subnet=172.18.0.0/24 tooling_app_network`

![creating-a-bridge-network-dedicated-for-our-project-to-run-Mysql-and-the-app](./Images/creating-a-bridge-network-dedicated-for-our-project-to-run-Mysql-and-the-app.png)

### Now we are going to run the MYSQL Server in a Container on the Bridge Network we created for our application and create an environment variable to store our Mysql root password

`docker run --network tooling_app_network -h mysqlserverhost --name=mysql-server -e MYSQL_ROOT_PASSWORD=banky12345  -d mysql/mysql-server:latest`

![running-our-mysql-image-in-a-container-on-the-bridge-network-we-created](./Images/running-our-mysql-image-in-a-container-on-the-bridge-network-we-created.png)

### Mysql script to create a new user to connect to the Mysql server remotely instead of using the root user

` " CREATE USER 'banky'@'%' IDENTIFIED BY 'banky12345';
GRANT ALL PRIVILEGES ON * . * TO 'banky'@'%'; " `

![mysql-script-to-create-a-new-user](./Images/mysql-script-to-create-a-new-user.png)

### Exporting our root password MYSQL_PW environment variable for the container to be able to access it

`export MYSQL_PW='banky12345'`

![root-password-exported](./Images/root-password-exported.png)

### Testing our new user access to the container

`docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < ./create_user.sql`

![root-password-exported](./Images/new-user-access-to-the-container.png)