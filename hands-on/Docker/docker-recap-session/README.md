# Hands-on Docker-Recap : Docker Review

Purpose of the this hands-on training is to review and recap of Docker Sessions

## Learning Outcomes

At the end of the this hands-on training, students will be able to;

- use Docker commands.

- run Docker containers on EC2 instance.

- manage Docker Volumes.

- manage Docker Networks.

- use Docker-compose tool.

## Part 1 - Launch a Docker Machine Instance and Connect with SSH

- Launch a Docker machine on Amazon Linux 2 AMI with security group allowing SSH connections using the [Cloudformation Template for Docker Machine Installation](../docker-01-installing-on-ec2-linux2/docker-installation-template.yml).

- Connect to your instance with SSH.

```bash
ssh -i .ssh/yasin.pem ec2-user@ec2-3-133-106-98.us-east-2.compute.amazonaws.com
```

## Part 2 - Basic Container Commands of Docker

- Check if the docker service is up and running.

```bash
sudo systemctl status docker
```

- Download and run `ubuntu` os with interactive shell open.

```bash
docker run -i -t ubuntu
```

- Display the os name on the container for the current user.

```bash
cat /etc/os-release
```

- Display the shell name on the container for the current user.

```bash
echo $0
```

- Update and upgrade os packages on `ubuntu` container.

```bash
apt-get update && apt-get upgrade -y
```

- Show the list of all containers available on Docker machine and explain container properties.

```bash
exit
docker ps -a
```

- Run the second `ubuntu` os with interactive shell open and name container as `yasin` and show that this `ubuntu` container is different from the previous one.

```bash
docker run -i -t --name yasin ubuntu
```

- Exit the `ubuntu` container and return to ec2-user bash shell.

```bash
exit
```

- Show the list of all containers again and explain the second `ubuntu` containers' properties and how the names of containers are given.

```bash
docker ps -a
```

- Restart the first container by its `ID`.

```bash
docker start 4e6
```

- Show only running containers and explain the status.

```bash
docker ps
```

- Stop the first container by its `ID` and show it is stopped.

```bash
docker stop 4e6 && docker ps -a
```

- Restart the `yasin` container by its name and list only running containers.

```bash
docker start yasin && docker ps
```

- Connect to the interactive shell of running `yasin` container and `exit` afterwards.

```bash
docker attach yasin
```

- Show that `yasin` container has stopped by listing all containers.

```bash
docker ps -a
```

- Restart the first container by its `ID` again and attach to it to show that the file we have created is still there under the home folder, and exit afterwards.

```bash
docker start 4e6 && docker attach 4e6
```

- Delete the first container using its `ID`.

```bash
docker rm 4e6
```

- Delete the second container using its name.

```bash
docker rm yasin
```

- Show that both of containers are not listed anymore.

```bash
docker ps -a
```
## Part 3 : Handling Docker Volumes:

- Run an `alpine` container with interactive shell open, and add command to run alpine shell. Here, explain explain what the alpine container is and why it is so popular. (Small size, Secure, Simple, Fast boot)

```bash
docker run -it alpine ash


- Create a file named `short-life.txt` under `/home` folder

```bash
cd home && touch short-life.txt && ls

- Exit the container and return to ec2-user bash shell.

```bash
exit
```

- Show the list of all containers available on Docker machine.

```bash
docker ps -a
```

- Start the alpine container and connect to it.

```bash
docker start 737 && docker attach 737
```

- Show that the file `short-life.txt` is still there, and explain why it is there. (Container holds it data until removed).

```bash
ls /home 
```

- Exit the container and return to ec2-user bash shell.

```bash
exit
```

- Remove the alpine container.

```bash
docker rm 737
```

- Show the list of all containers, and the alpine container is gone with its data.

```bash
docker ps -a

- Explain why we need volumes in Docker.

- List the volumes available in Docker, since not added volume before list should be empty.

```bash
docker volume ls
```

- Create a volume named `cw-vol`.

```bash
docker volume create cw-vol
```

- List the volumes available in Docker, should see local volume `cw-vol` in the list.

```bash
docker volume ls
```

- Show details and explain the volume `cw-vol`. Note the mount point: `/var/lib/docker/volumes/cw-vol/_data`.

```bash
docker volume inspect cw-vol
```

- List all files/folders under the mount point of the volume `cw-vol`, should see nothing listed.

```bash
sudo ls -al  /var/lib/docker/volumes/cw-vol/_data
```

- Run a `alpine` container with interactive shell open, name the container as `yasin`, attach the volume `cw-vol` to `/cw` mount point in the container, and add command to run alpine shell. Here, explain `--volume` and `v` flags.

```bash
docker run -it --name yasin -v cw-vol:/cw alpine ash
```

- List files/folder in `yasin` container, show mounting point `/cw`, and explain the mounted volume `cw-vol`.

```bash
ls
```

- Create a file in `yasin` container under `/cw` folder.

```bash
cd cw && echo "This file is created in the container yasin" > i-will-persist.txt
```

- List the files in `/cw` folder, and show content of `i-will-persist.txt`.

```bash
ls && cat i-will-persist.txt
```

- Exit the `yasin` container and return to ec2-user bash shell.

```bash
exit
```

- Show the list of all containers available on Docker machine.

```bash
docker ps -a
```

- Remove the `yasin` container.

```bash
docker rm yasin
```

- Show the list of all containers, and the `yasin` container is gone.

```bash
docker ps -a
```

- List all files/folders under the volume `cw-vol`, show that the file `i-will-persist.txt` is there.

```bash
sudo ls -al  /var/lib/docker/volumes/cw-vol/_data && sudo cat /var/lib/docker/volumes/cw-vol/_data/i-will-persist.txt

# Using Same Volume with Different Containers:

- Run a `alpine` container with interactive shell open, name the container as `yasin2nd`, attach the volume `cw-vol` to `/cw2nd` mount point in the container, and add command to run alpine shell.

```bash
docker run -it --name yasin2nd -v cw-vol:/cw2nd alpine ash
```

- List the files in `/cw2nd` folder, and show that we can reach the file `i-will-persist.txt`.

```bash
ls -l /cw2nd && cat /cw2nd/i-will-persist.txt
```

- Create an another file in `yasin2nd` container under `/cw2nd` folder.

```bash
cd cw2nd && echo "This is a file of the container yasin2nd" > loadmore.txt
```

- List the files in `/cw2nd` folder, and show content of `loadmore.txt`.

```bash
ls && cat loadmore.txt
```

- Exit the `yasin2nd` container and return to ec2-user bash shell.

```bash
exit
```

- Run a `ubuntu` container with interactive shell open, name the container as `yasin3rd`, attach the volume `cw-vol` to `/cw3rd` mount point in the container, and add command to run bash shell.

```bash
docker run -it --name yasin3rd -v cw-vol:/cw3rd ubuntu bash
```

- List the files in `/cw3rd` folder, and show that we can reach the all files created earlier.

```bash
ls -l /cw3rd
```

- Create an another file in `yasin3rd` container under `/cw3rd` folder.

```bash
cd cw3rd && touch file-from-3rd.txt && ls
```

- Exit the `yasin3rd` container and return to ec2-user bash shell.

```bash
exit


## Part 4 : Handling Docker Networking:


- List all networks available in Docker, and explain types of networks.

```bash
docker network ls
```

- Run two `alpine` containers with interactive shell, in detached mode, name the container as `yasin1st` and `yasin2nd`, and add command to run alpine shell. Here, explain what the detached mode means.

```bash
docker container run -dit --name yasin1st alpine ash
docker container run -dit --name yasin2nd alpine ash
```

- Show the list of running containers on Docker machine.

```bash
docker ps
```

- Show the details of `bridge` network, and explain properties (subnet, ips) and why containers are in the default network bridge.

```bash
docker network inspect bridge | less
```

- Get the IP of `yasin2st` container.

```bash
docker container inspect yasin2nd | grep IPAddress
```

- Connect to the `yasin1st` container.

```bash
docker container attach yasin1st
```

- Show the details of network interface configuration of `yasin1st` container.

```bash
ifconfig
```

- Open an other terminal and connect your ec2 instance. Show the details of network interface configuration of ec2 instance. 

```bash
ifconfig
```

- Compare with two configurations.

- In the `yasin1st` container ping google.com four times to check internet connection.

```bash
ping -c 4 google.com
```

- Ping `yasin2nd `container by its IP four times to show the connection.

```bash
ping -c 4 172.17.0.3
```

- Try to ping `yasin2nd `container by its name, should face with bad address. Explain why failed (due to default bridge configuration not works with container names)

```bash
ping -c 4 yasin2nd
```

- Stop and delete the containers

```bash
docker container stop yasin2nd
docker container rm yasin1st yasin2nd
```

# User-defined Network Bridge in Docker:

- Create a bridge network `yasinnet`.

```bash
docker network create --driver bridge yasinnet
```

- List all networks available in Docker, and show the user-defined `yasinnet`.

```bash
docker network ls
```

- Show the details of `yasinnet`, and show that there is no container yet.

```bash
docker network inspect yasinnet
```

- Run four `alpine` containers with interactive shell, in detached mode, name the containers as `yasin1st`, `yasin2nd`, `yasin3rd` and `yasin4th`, and add command to run alpine shell. Here, 1st and 2nd containers should be in `yasinnet`, 3rd container should be in default network bridge, 4th container should be in both `yasinnet` and default network bridge.

```bash
docker container run -dit --network yasinnet --name yasin1st alpine ash
docker container run -dit --network yasinnet --name yasin2nd alpine ash
docker container run -dit --name yasin3rd alpine ash
docker container run -dit --name yasin4th alpine ash
docker network connect yasinnet yasin4th
```

- List all running containers and show there up and running.

```bash
docker container ls
```

- Show the details of `yasinnet`, and explain newly added containers. (1st, 2nd, and 4th containers should be in the list)

```bash
docker network inspect yasinnet
```

- Show the details of  default network bridge, and explain newly added containers. (3rd and 4th containers should be in the list)

```bash
docker network inspect bridge
```

- Connect to the `yasin1st` container.

```bash
docker attach yasin1st
```

- Ping `yasin2nd` and `yasin4th` container by its name to show that in user-defined network, container names can be used in networking.

```bash
ping -c 4 yasin2nd
ping -c 4 yasin4th
```

- Try to ping `yasin3rd` container by its name and IP, should face with bad address because 3rd container is in different network.

```bash
ping -c 4 yasin3rd
ping -c 4 172.17.0.2
```

- Exit the `yasin1st` container without stopping and return to ec2-user bash shell.

- Connect to the `yasin4th` container, since it is in both network should connect all containers.

```bash
docker container attach yasin4th
```

- Ping `yasin2nd` and `yasin1st` container by its name, ping `yasin3rd` container with its IP. Explain why used IP, instead of name.

```bash
ping -c 4 yasin1st
ping -c 4 yasin2nd
ping -c 4 172.17.0.2
```

- Exit from `yasin4th` container. Stop and remove all containers.

```bash
docker container stop yasin1st yasin2nd yasin3rd yasin4th
docker container rm yasin1st yasin2nd yasin3rd yasin4th
```

- Delete `yasinnet` network

```bash
docker network rm yasinnet

# Part-5: Docker Compose Operations:

```
docker-compose --version
```

```bash
mkdir composetest
cd composetest
```

- Create a file called `app.py` in your project folder and paste the following python code in it. In this example, the application uses the Flask framework and maintains a hit counter in Redis, and  `redis` is the hostname of the `Redis container` on the application’s network. We use the default port for Redis, `6379`.

```python
import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)


def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)


@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
```

- Create another file called `requirements.txt` in your project folder, add `flask` and `redis` as package list.

```bash
echo '
flask
redis
' > requirements.txt
```

- Create a Dockerfile which builds a Docker image and explain what it does.

```text
The image contains all the dependencies for the application, including Python itself.
1. Build an image starting with the Python 3.7 image.
2. Set the working directory to `/code`.
3. Set environment variables used by the flask command.
4. Install gcc and other dependencies
5. Copy `requirements.txt` and install the Python dependencies.
6. Add metadata to the image to describe that the container is listening on port 5000
7. Copy the current directory `.` in the project to the workdir `.` in the image.
8. Set the default command for the container to flask run.
```

```bash
echo '
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP app.py
ENV FLASK_RUN_HOST 0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run"]
' > Dockerfile
```

- Create a file called `docker-compose.yml` in your project folder and define services and explain services.

```text
This Compose file defines two services: web and redis.

### Web service
The web service uses an image that’s built from the Dockerfile in the current directory. It then binds the container and the host machine to the exposed port, 5000. This example service uses the default port for the Flask web server, 5000.

### Redis service
The redis service uses a public Redis image pulled from the Docker Hub registry.
```

```bash
echo '
version: "3"
services:
  web:
    build: .
    ports:
      - "5000:5000"
  redis:
    image: "redis:alpine"
' > docker-compose.yml
```

- Build and run your app with `Docker Compose` and explains what is happening.

```text
Docker compose pulls a Redis image, builds an image for our the app code,
and starts the services defined. In this case, the code is statically copied into the image at build time.
```

```bash
docker-compose up
```

- Add a rule within the security group of Docker Instance allowing TCP connections through port `5000` from anywhere in the AWS Management Console.

```text
Type        : Custom TCP
Protocol    : TCP
Port Range  : 5000
Source      : 0.0.0.0/0
Description : -
```

- Enter http://`ec2-host-name`:5000/ in a browser to see the application running.


- Open an other terminal and connect to EC2 instance. 