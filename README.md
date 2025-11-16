 
# Flask App with MySQL Docker Setup

This is a simple Flask app that interacts with a MySQL database. The app allows users to submit messages, which are then stored in the database and displayed on the frontend.

<img width="1232" height="389" alt="image" src="https://github.com/user-attachments/assets/c4624ad0-8e94-408b-9b26-c00aea727cdb" />

## Prerequisites

Before you begin, make sure you have the following installed:

- Docker
- Git (optional, for cloning the repository)

## Setup

1. Clone this repository (if you haven't already):

   ```bash
   git clone https://github.com/your-username/your-repo-name.git
   ```

2. Navigate to the project directory:

   ```bash
   cd your-repo-name
   ```

3. Create a `.env` file in the project directory to store your MySQL environment variables:

   ```bash
   touch .env
   ```

4. Open the `.env` file and add your MySQL configuration:

   ```
   MYSQL_HOST=mysql
   MYSQL_USER=your_username
   MYSQL_PASSWORD=your_password
   MYSQL_DB=your_database
   ```

## Begineer Guid

1. Docker Installation .

```
sudo apt install docker.io
```


2. Build Docker-image form Dockerfile
```
docker build -t flaskapp .
```
3. Check image is build
```
docker images
```

4. Create a netowk to have communication between the flaskapp and mysql container
 
 ```
 docker network create twotire

 ```
5. Now run the Containers
```
docker run -d  -p 3306:3306 --name=mysql  --network=twotire   -e MYSQL_ROOT_PASSWORD=admin   -e MYSQL_USER=admin   -e MYSQL_PASSWORD=admin   -e MYSQL_DATABASE=myDB   mysql:5.7
docker run -d -p 5000:5000 --name=flaskapp  --network=twotire   -e MYSQL_HOST=mysql   -e MYSQL_USER=admin   -e MYSQL_PASSWORD=admin   -e MYSQL_DB=myDB   flaskapp:latest


```
6. Check is the applicaiton is runing on `PbulicIP:5000`.  if security group is not configured then edit its inboud rule for port 5000.

7. To go inside mysql container take container ID from `docker ps` then

```
docker exec -it CONTAINER-ID bash
```

-  for mysql container

```
mysql -u root -p
```
-  Now  to see show databases

```
show databases;
```

-  to do inside a database 

```

use myDB; # or your database name 

```

-  then to list the tables inside that database

```
show tables;
```



## Usage
1. Start the containers using Docker Compose:

```
 docker-compose up --build
 ```

2. Access the Flask app in your web browser:

   - Frontend: http://localhost
   - Backend: http://localhost:5000

3. Create the `messages` table in your MySQL database:

   - Use a MySQL client or tool (e.g., phpMyAdmin) to execute the following SQL commands:
   
     ```sql
     CREATE TABLE messages (
         id INT AUTO_INCREMENT PRIMARY KEY,
         message TEXT
     );
     ```

4. Interact with the app:

   - Visit http://localhost to see the frontend. You can submit new messages using the form.
   - Visit http://localhost:5000/insert_sql to insert a message directly into the `messages` table via an SQL query.

## Cleaning Up

To stop and remove the Docker containers, press `Ctrl+C` in the terminal where the containers are running, or use the following command:

```bash
docker-compose down
```

## To run this two-tier application using  without docker-compose

- First create a docker image from Dockerfile
```bash
docker build -t flaskapp .
```

- Now, make sure that you have created a network using following command
```bash
docker network create twotier
```

- Attach both the containers in the same network, so that they can communicate with each other

i) MySQL container 
```bash
docker run -d \
    --name mysql \
    -v mysql-data:/var/lib/mysql \
    --network=twotier \
    -e MYSQL_DATABASE=mydb \
    -e MYSQL_ROOT_PASSWORD=admin \
    -p 3306:3306 \
    mysql:5.7

```
ii) Backend container
```bash
docker run -d \
    --name flaskapp \
    --network=twotier \
    -e MYSQL_HOST=mysql \
    -e MYSQL_USER=root \
    -e MYSQL_PASSWORD=admin \
    -e MYSQL_DB=mydb \
    -p 5000:5000 \
    flaskapp:latest

```

## Notes

- Make sure to replace placeholders (e.g., `your_username`, `your_password`, `your_database`) with your actual MySQL configuration.

- This is a basic setup for demonstration purposes. In a production environment, you should follow best practices for security and performance.

- Be cautious when executing SQL queries directly. Validate and sanitize user inputs to prevent vulnerabilities like SQL injection.

- If you encounter issues, check Docker logs and error messages for troubleshooting.

```

