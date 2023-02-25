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

###Task3: Ensure apps runs locally.
Building container image.
```
docker build -t  backend-flask ./backend-flask
```

