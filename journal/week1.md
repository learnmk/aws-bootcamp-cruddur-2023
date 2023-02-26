# Week 1 — App Containerization

### Task1: Configure VScode Extension.

Docker extension is preinstalled in Gitpod.

### Task2: Explore Codebase

Backend Dockerfile
```dockerfile
FROM python:3.10-slim-buster

#Working Directory of the running docker.
WORKDIR /backend-flask

#Copying required plugins files to container and installing it.
COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

COPY . .

#Setting up environment variable inside container.
ENV FLASK_ENV=development

#Exposing container port on host.
EXPOSE ${PORT}

#command to run after conatainer startup.
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]

```

Frontend Dockerfile
```dockerfile
FROM node:16.18

#Setting up environment variable inside container.
ENV PORT=3000

#Copying code to container.
COPY . /frontend-react-js
WORKDIR /frontend-react-js

# Run creates a writeable container layer over the specified image, and then starts it using the specified command
RUN npm install

EXPOSE ${PORT}
CMD ["npm", "start"]

```

### Task3: Ensure apps runs locally.
+ Building container image for backend.
```
docker build -t  backend-flask ./backend-flask
```
+ Running backend container
```
docker container run --rm -p 4567:4567 -d backend-flask
```
Making sure container is running and port is open in Gitpod.

![Screenshot 2023-02-25 at 2 33 15 PM](https://user-images.githubusercontent.com/125124581/221348652-21532a54-1375-4ec7-b628-2a882369f9eb.png)


+ Building container image for frontend.
```
docker build -t  front-react-js ./front-react-js
```
+ Running backend container
```
docker container run --rm -p 3000:3000 -d frontend-react-js
```
Making sure frontend container is running.
![Screenshot 2023-02-25 at 2 42 16 PM](https://user-images.githubusercontent.com/125124581/221348977-7525e15c-81cf-4f8b-a4e3-111cd683d7cf.png)


### Task4: Creating Docker Compose file.
Created Docker compose file in the root directory. Added four containers in the compose file.
+ backend-flask
+ frontend-react-js
+ postgresdb
+ dynamodb

```
docker compose -f "docker-compose.yml" up -d --build 
```

```dockerfile

version: "3.8"
services:
  backend-flask:
    environment:
      FRONTEND_URL: "https://3000-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
      BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./backend-flask
    ports:
      - "4567:4567"
    volumes:
      - ./backend-flask:/backend-flask
  frontend-react-js:
    environment:
      REACT_APP_BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./frontend-react-js
    ports:
      - "3000:3000"
    volumes:
      - ./frontend-react-js:/frontend-react-js
  db:
    image: postgres:13-alpine
    restart: always
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
    ports:
      - '5432:5432'
    volumes: 
      - db:/var/lib/postgresql/data
    dynamodb-local:
    # https://stackoverflow.com/questions/67533058/persist-local-dynamodb-data-in-volumes-lack-permission-unable-to-open-databa
    # We needed to add user:root to get this working.
    user: root
    command: "-jar DynamoDBLocal.jar -sharedDb -dbPath ./data"
    image: "amazon/dynamodb-local:latest"
    container_name: dynamodb-local
    ports:
      - "8000:8000"
    volumes:
      - "./docker/dynamodb:/home/dynamodblocal/data"
    working_dir: /home/dynamodblocal

networks: 
  internal-network:
    driver: bridge
    name: cruddur
```

### Task5: Install postgres client in gitpod updated gitpod.yml

```sh
  - name: postgres
    init: |
      curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
      echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
      sudo apt update
      sudo apt install -y postgresql-client-13 libpq-dev
```

Tested connection to postgres.

![Screenshot 2023-02-25 at 3 38 07 PM](https://user-images.githubusercontent.com/125124581/221351424-6ffcb877-22d6-4f6f-9ec1-35cd434eeabe.png)

### Task:6 Updated OpenAPI to add notification endpoint.

![Screenshot 2023-02-25 at 6 20 03 PM](https://user-images.githubusercontent.com/125124581/221357884-0a327194-0492-4ec6-97fd-68ce5b557f21.png)

Manage to create user and see notifications page

![Screenshot 2023-02-26 at 10 52 21 AM](https://user-images.githubusercontent.com/125124581/221408477-c993b04a-7ca2-4b1e-9a6f-2780b772dd3b.png)

# Homework Challenge

### Push and Tag image to DockerHub

Created new account in Dockerhub. Tag image and push it into Dockerhub.

```sh
docker image tag aws-bootcamp-cruddur-2023-frontend-react-js:latest learnkhaire/cruddur:latest
docker push learnkhaire/cruddur:latest
```

![Screenshot 2023-02-26 at 5 40 32 PM](https://user-images.githubusercontent.com/125124581/221409709-61082ff1-4a11-477e-a2dc-fbb2bf200ee8.png)

### Learn how to install Docker on your localmachine and get the same containers running outside of Gitpod

![Screenshot 2023-02-26 at 5 58 01 PM](https://user-images.githubusercontent.com/125124581/221410533-d563b9e0-196c-4811-95fe-ad337a8bdd11.png)
![Screenshot 2023-02-26 at 5 57 45 PM](https://user-images.githubusercontent.com/125124581/221410543-32160787-951d-4ae6-9070-646ac11b4e4b.png)


### Launch an EC2 instance that has docker installed, and pull a container to demonstrate you can run your own docker processes. 

Added below commands in user data during launching of EC2 instance.
```sh
#!/bin/bash
sudo yum update -y
sudo amazon-linux-extras install docker
sudo service docker start
sudo systemctl enable docker
sudo systemctl enable docker
```

![Screenshot 2023-02-26 at 6 10 54 PM](https://user-images.githubusercontent.com/125124581/221411469-f6d2fd19-0f0f-4098-a3e3-fb9c9df45358.png)

