# Node.JS app connected to databse using docker containers
JS page connected to MongoDB container

### This is a demonstration for created a Javascript container and connecting it with Mongo database through docker-compose.

Requirements:
1. Docker engine
2. Mongo engine
3. Mongo express - for interface
4. Web app running through node JS.

# Introduction:
First, we'll create a dockerfile that will create a container for the Node JS server. This is how the file will look like:

```

FROM node:20-alpine

ENV MONGO_DB_USERNAME=admin\
    MONGO_DB_PWD=password

RUN mkdir -p /home/app

COPY ./app /home/app

RUN cd /home/app && npm install


CMD ["node", "/home/app/server.js"]

```

Where the first line installs the node image from docker hub. The ENV registers environmental variables, each image has its own defined variables. Then the web server apps located in the app direcroty will be copied to the directory '/home/app'.

# Build the image

Browse through the image directory and build the docker image with the following command:

```
docker build -t app:1.0 .
```
# Upload the image to repository
Using ECR servide in AWS cloud, create a repository, then follow the instructions shown in the 'view push commands' to tag & push the image to the repository.

# Create a docker compose file

Now to let the app work with a database, we will link it with the database container using a docker compose file 'mongoapp.yaml' :

```
version: '3'
services:
  app:                                  # Container label upon installation
    image: ${docker-registry}/app:1.0   # Your docker image URL uploaded in ECR
    ports:
      - 3000:3000                       # The ports you want to link to the container as of Host-port:Docker-Port
      
  # Second containter
  mongodb:                              
    image: mongo                        
    ports:
      - 27017:27017
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password
    volumes:                            # Setting a volume storage that will store the database data in the container's and link it to a volume in the host.
      - mongo-data:/data/db             # Name_of_host_volume:Directory_in_Container's_virtual_host
 
 # Second containter
  mongo-express:
    image: mongo-express
    restart: always                     # It assures the connectivity between the database and the database's interface
    ports:
      - 8080:8081
    environment:
      - ME_CONFIG_MONGODB_ADMINUSERNAME=admin           # Database's username identified earlier
      - ME_CONFIG_MONGODB_ADMINPASSWORD=password        # Database's password
      - ME_CONFIG_MONGODB_SERVER=mongodb                # Here's where you link the container with the databse by specifying the container's name.
volumes:                                # This creates a local volume storage that syncs the data in the container to ' mongo-data' in the local host and vice vera.
  mongo-data:
    driver: local

```

Now to run the docker-compose file to install the containers:

```
docker-compose -f mongoapp.yaml up    # Down to delete the containers.
```

This will run the containersin one interactive network environment that can be show by:
```
docker network ls
```

To make sure of running containers:

```
docker ps -a
```

