--- 
title: "Deploying RASA X on server with Docker Compose"
description: "Instructions along with resolution of bugs that I faced"
layout: post
toc: true
comments: true
hide: false
search_exclude: true
categories: [RASA]
author: Vansh Kapil
metadata_key1: metadata_value1
metadata_key2: metadata_value2
---


Install RASA X on server using Docker Compose 
We'll follow the instructions [here](https://rasa.com/docs/rasa-x/installation-and-setup/install/docker-compose)

### Creating a VM on Google Cloud

Follow the Hardware and OS requirements 

- E2 Medium
- Ubuntu 16
- 4gb ram and 20gb HDD

Login in the terminal 

### Install RASA X

```jsx
curl -sSL -o install.sh https://storage.googleapis.com/rasa-x-releases/0.33.2/install.sh
```

```jsx
sudo bash ./install.sh
```

Get root access 

```jsx
sudo -i
```

Startup RASA X

```jsx
cd /etc/rasa
sudo docker-compose up -d
```

Get access and setup passwords 

```jsx
sudo python rasa_x_commands.py create --update admin me <PASSWORD>
```

- If you get error

```jsx
File "rasa_x_commands.py", line 102
    command = f"delete {args.username}"
```

find the container name of your RASA X container 

```jsx
docker ps 
```

it may look like **rasa_rasa-x_1**

Run the following command to enter the bash of that container, so you can execute a command inside the container

```jsx
docker exec -it rasa_rasa-x_1 /bin/bash
```

You should get access to container bash which may look like **rasa@d2833a965284 :~$**

Execute the following command to create your password 

```jsx
python3 /app/scripts/manage_users.py create me password123 admin --update
```

password123 is the password in this case, you should select a stronger one. 

**exit** the bash 

Check if your password is created by opening RASA X(click on the external IP in google cloud panel) and login in using the password you just created

### Connect your github repo

Follow the instructions [here](https://rasa.com/docs/rasa-x/installation-and-setup/deploy#integrated-version-control)

**Train a model using RASA X interface** 

### Manually building Action Server

Prepare the action files.

- Create an actions folder inside /etc/rasa

```jsx
mkdir actions
```

- Create all the action server related files in actions folder. To create a file

```jsx
nano actions.py
```

and paste the contents of the file

**Create a Dockerfile** 

inside /etc/rasa directory 

create a file **Dockerfile**

```jsx
nano Dockerfile
```

copy the following contents in it

```jsx
# Extend the official Rasa SDK image
FROM rasa/rasa-sdk:2.0.0rc1

# Use subdirectory as working directory
WORKDIR /app

# Copy any additional custom requirements, if necessary (uncomment next line)
COPY actions/requirements-actions.txt ./

# Change back to root user to install dependencies
USER root

# Install extra requirements for actions code, if necessary (uncomment next line)
RUN pip install -r requirements-actions.txt

# Copy actions folder to working directory
COPY ./actions /app/actions

# By best practices, don't run the code with root user
USER 1001
```

Build a custom image

```jsx
docker build . -t <account_username>/<repository_name>:<custom_image_tag>
```

- If you get error

```jsx
Sending build context to Docker daemon  124.3MB
Step 1/7 : FROM rasa/rasa-sdk:2.1.0
manifest for rasa/rasa-sdk:2.1.0 not found: manifest unknown: manifest unknown
```

Check the rasa-sdk version for compatibility in [compatibility matrix](https://rasa.com/docs/rasa-x/changelog/compatibility-matrix/)  , remember we installed RASA X version 0.33.2 

Make the change in Dockerfile

At the time of this doc, 2.0.0rc1 worked 

once the image is created successfully , login docker hub and push the image

```jsx
docker login --username <account_username> --password <account_password>
docker push <account_username>/<repository_name>:<custom_image_tag>
```

### Connect the Custom Action server

Follow the instructions [here](https://rasa.com/docs/rasa-x/installation-and-setup/customize/#connecting-a-custom-action-server)

create a docker-compose.override.yml 

```jsx
version: '3.4'
services:
  app:
    image: '<account_username>/<repository_name>:<custom_image_tag>'
    volumes:
      - './actions:/app/actions'
    expose:
      - '5055'
    depends_on:
      - rasa-production
```

Restart the docker-compose 

```jsx
cd /etc/rasa
sudo docker-compose down
sudo docker-compose up -d
```

### Securing the server with certbot

Connect you domain name to external IP first

```jsx
docker-compose down
sudo certbot certonly
```

if it says 

```jsx
sudo: certbot: command not found
```

run 

```jsx
sudo snap install --classic certbot
sudo certbot certonly
```

Follow the instructions [here](https://rasa.com/docs/rasa-x/installation-and-setup/customize/#securing-with-ssl)

### Confirm Action Server is runing

```jsx
Sudo docker ps
```

Action server is named **rasa_app_1** ; check the status, it should not be restarting all the time.

If you find any of the container to be restarting you need to check logs 

```jsx
docker-compose logs app 

docker-compose logs rasa-x
```