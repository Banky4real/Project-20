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

![new-user-access-to-the-container](./Images/new-user-access-to-the-container.png)

### Connecting to the Mysql server from the second container that has the mysql client running on it. We pulled down the mysql client image from docker hub on the tooling-app bridge network we created and connecting to the mysql-server.

`docker run --network tooling_app_network --name mysql-client -it --rm mysql mysql -h mysqlserverhost -u banky -p`

![connecting-to-mysql-server-from-my-sql-client-on-a-second-container-running-my-sql-client](./Images/connecting-to-mysql-server-from-my-sql-client-on-a-second-container-running-my-sql-client.png)

### Preparing a Database schema that our tooling application will connect to for data storage.

### Cloning down our tooling app.

`git clone https://github.com/dareyio/tooling.git`

![cloning-our-tooling-app-from-github-repo](./Images/cloning-our-tooling-app-from-github-repo.png)

### Exporting the location of our sql file for the container to be able to access it.

`export tooling_db_schema=~/tooling/html/tooling_db_schema.sql`

![exporting-the-location-of-the-sql-file](./Images/exporting-the-location-of-the-sql-file.png)

![location-exported](./Images/location-exported.png)


### Executing a command in the running container.

`docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < $tooling_db_schema
`
![executing-a-command-in-the-running-container](./Images/executing-a-command-in-the-running-container.png)

## Containerizing our tooling app.

### Building our Dockerfile which has been updated with the latest version of php, which is the base image

`docker build . -t tooling:0.0.1`

![Building-our-Dockerfile-into-an-image](./Images/Building-our-Dockerfile-into-an-image.png)

### Running our Docker image which we have just built in a container on the same bridge network with the database so they can communicate with each other.

`docker run --name=banky_tooling-app --network tooling_app_network -p=8085:80 -d -it tooling:0.0.1 bash`

`docker ps`

![Docker-image-currently-running](./Images/Docker-image-currently-running.png)

### Tooling App Live on port 8085 after starting apache2 server in the container and some troubleshooting.

`docker ps`

![tooling-app-live-on-port-8085](./Images/tooling-app-live-on-port-8085.png)

![tooling-app-dashboard](./Images/tooling-app-dashboard.png)

![Environment-tab-of-our-tooling-app](./Images/Environment-tab-of-our-tooling-app.png)

## Practice Task
