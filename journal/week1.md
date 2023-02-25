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

