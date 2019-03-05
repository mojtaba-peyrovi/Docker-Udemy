### Docker Mastery course from Udemy:
---
my code github:

teacher's github for this course: https://github.com/bretfisher/udemy-docker-mastery

- Docker-machine starts on VirtualBox when we install it. 


##### 01- course introduction and Docker setup:

```
docker-machine ls
```
it returns the list of virtual machines made by docker in Virtualbox.

__IMPORTANT:__ code paths enabled for Bind Mounts only work in C:/Users folder

 ##### 02 - Creating and using containers like a boss:
 
 The first thing we need to work with is Containers. not making images or advanced stuff.
 
 __Docker Engine and Daemon are the same.__
 
 ---
 

##### Difference between Image and Container:

Image is a bunch of libraries and source-codes that the application is made bby them, whereas a container is an instance of the image.

You can have many containers all based on the same image.

__Docker Registries:__ something similar to github repositories that we can push our containers to and pull them. 

###### running an Nginx container:
```
~ docker container run --publish 80:80 nginx
```
docker downloads an instance of nginx image and starts a new container from that image.

when we want to have the terminal work normally we can have the container run in the background. for doing this we need to ad -d to the command.

- when we want to stop a container, we will say:
```
docker contianer stop <2_or_3_first_digits_of_the_container_id>
```
We can also assign a name to the container:
```
docker container run --name <name>
```
We can delete a container by saying:
```
docker contianer rm -f <contianer_name_or_code>
```
- What happens when we run a new container?
first it will search locally in image cache to see if it exists there, if yes, it pulls it locally, if not it pulls it from hub.docker which is the online repository.
then it will give it a virtual IP on a private network inside the docker engine, and if we specify a port for it, it will assign a port to it othewise it doesn't forward it to any port.
Finally, the container starts by using the CMD inside the image Dockerfile.

When we want to have multiple containers, each one of them has to have its own port. They can't share ports.

- making a mysql container:
```
docker run -d -p 3306:3306 --name mysql-moji -e MYSQL_RANDOM_ROOT_PASSWORD=yes mysql
```

MYSQL_RANDOM_ROOT_PASSWORD makes random passwords each time we run the container. In order to see the random password we can run:
```
docker container logs <mysql_container_name>
```
Now we make a httpd container:
```
docker run -d --name webserver-moji -p 8080:80 httpd
```
And for making a nginx container we say:
```
docker run -d --name proxy-moji -p 80:80 nginx
```
We can check if they work, without going to the browser by saying:
```
curl 192.168.99.100:8080
for httpd
curl 192.186.99.100
for nginx
and mysql doesn't have any port allocated.
```
When I want to close them I say:
```
docker stop proxy-moji webserver-moji mysql-moji
```
This command shows all containers including stopped ones:
```
docker ps -a
```
This deletes all stopped containers:
```
docker rm $(docker ps -a -q)
```

and this stops all running containers:
```
docker stop $(docker ps -a -q)
```
A good command that shows a livestream info about each container is:
```
docker container stats
```
especially cpu usage is useful to make sure none of them is exhausting the system.

---
Sometimes we want to run a container, but we want to have a shell inside it and want to run second process inside it. For this we need to add more tags:
```
docker run -it --name proxy nginx bash
```
-i means interactive, and -t means TTY. after nginx we can have extra commands, we passed bash which means terminal access.
The prompt shows root@<container_id>

From this command line we can access all files inside the container, by running:
```
ls -al
```   
and we can go to directories, add, remove, edit them. And to get out of the shell we just type exit.

__Alpine:__ A very small Linux distribution that is only 5 MB.

When we run an Alpine docker image, we can't run bash for it, becuase it is so small and bash doesn't come with it. Instead we can use sh like this:
```
docker run -it alpine sh
``` 
But if we want bash its package manager is apk and we can install bash with apk inside alpine.

---
#### Networks:

When we have mysql and php/apache in one container, they don't need their ports to be specified. they find each other. Or node.js and mongodb. (without -p)

__IMPORTANT:__ the template for -p (--publish) is always like this:
```
HOST:CONTAINER
```
If we want to see which port is being used by the container:
```
docker container port <container_name>
```
If I want to see the ip address of my container called mojimoji:
```
docker container inspect --format '{{ .NetworkSettings.IPAddress }}' mojimoji
```
 ##### 03 - Container images, where to find them and how to build them:
 
---
What is inside the image? app binaries and dependencies. and metadata about the image and how to run the image.

There is no OS inside the image, nor kernels or modules.

- Images use a concept called union file system to make layers about the changes.

We can see the history of image layers for each container like this:
```
docker history <image_name>
```
We can also see all the meta data of an image by running this:
```
docker image inspect <image_name>
```
We can see the exposed port for the image under ExposedPorts. and many other things like environment variables, their unique id called SHA, etc.

__Image tags and pushing to docker hub:__

Images don't have any name, instead they are known by 3 parameters:
```
<user>/<repo>:<tag>
```
Only official repos don't have any username. 

__Tags:__ when we see a repository on Docker hub we will see different tags. those written on the same line are all the same versions and we can use any of them. 

When we try to pull two images with the tags which are actually the same (written in same line on the website), they will be installed with the same image id and docker knows they are the same.

We can change the image  label of an existing image by:
```
docker image tag <source_image[:tag]> <target_image[:tag]>
```
for example: if I say "docker image tag nginx mopeyrovi/nginx" it will make a new image with this name and same image id.

This image is made on my computer but not at Docker hub. What we can do now is to push it to the hub as a new image:
```
docker push <new_iamge_name>
```
before that we need to login to docker hub. 

- __Dockerfile and creating images:__

Dockerfile (capital D) has different parts:

    1- FROM <image name>  example: FROM debian:jessie

Nowadays people use Alpine more than Debian because it is so much smaller and faster to run. 
If we want to have an empty container we can say FROM scratch.

    2- ENV to specify environmental variables of the image. example: ENV NGINX_VERSION 1.11.10-1~jessie

    3- RUN <a bunch of shell commands that will run when the container is being built>

Anytime in RUN we add && in the beginning of each line, it means that they all belong to the ssme layer and doesn't make new layers.

    4- EXPOSE 80 443  

Expose shows what ports should be exposed by the container. We allow the container to receive packets on these ports.
 
    5- CMD[ "command1" "command2" "command3"... ] 
   
  
It lists a bunch of commands should be executed when the container is run.

after making the Dockerfile we have to build the image based on that:
```
docker image build -t <tag_name> .   => the dot means build the image in this folder
```    
It will run the Dockerfile line by line. Each step of it will have a hash (code) that will be saved and anytime later if we want to add them again it wouldn't re-run.

Anytime we have any change on Dockerfile, we need to run docker build again and it will just make new layers for the new changes and for the old lines, it will run them from the cache.

We have additional lines at Dockerfile:

```
WORKDIR /usr/share/nginx/html  => if there is a html and nginx we can specify the location

COPY index.html index.html this line just copies everything from local machine to the container
```
When we use FROM we inherit everything from that image.


_Interesting:__ when we say docker run, we can use --rm flag which means delete it when we close it. 

 ##### 04 - Container lifetime and persistent data:
 
 - Containers are immutable and ephemeral. It means we can remove them anytime and create a new one using images.
 
  It means anytime we have to have any changes in a container or any update, we will re-deploy a new container.
  
  __Separation of Concerns:__ It is a concept in Docker that says you can update applications with creating new containers with an updated version of our app, and our unique data such as database data, etc. are where they have to be and we don't need to worry for them.
  
  __Persistent Data:__  These are the data our app needs to keep persistent adn not changed. For doing this, there are two ways "Volumes" and "Bind Mounts".
  
  __Volumes:__ Makes special location outside of container UFS. It allows cross container removal. We can attach volumes to any container we want.
  
  __Bind Mounts:__ links containers path to host path.
  
  Example: when we run a mysql image, in the Dockerfile there is a line for volume saying:
  ```
  VOLUME /var/lib/mysql   
```
It tells docker to create a volume in the mentioned location anytime we will create a new container from the image. It means if we put a file in this location inaside the container, will stay there until we manually delete the volume. 

__IMPORTANT:__ Volumes need manual deletion. They won't go away when we remove the container.

- We can see volumes on the machine like this:
```
docker volume ls
```
- Also we can get more info about a specific volume like this:
```
docker colume inspect <volume_id>
```
This is a great thing to preserve our data, because the database instances live outside the containers and we know they are safe.

We can also have named volumes by adding -v when we are going to make containers:
```
docker container run -d --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=True -v mysql_db:/var/lib/mysql mysql
```

What it does, is adding a volume named mysql_db in the mentioned location.

- when we want to make a new container using the same volume we can again mention it with -v and it won't make a new volume. it will just use the old one.

__Bind Mounting:__ maps a host file or directory to a container file or directory.

for Bind Mounting we need to use container run and we simply have to say:
```
docker continer run -v // User/mojiway/stuff:/path/container (Windows)
docker continer run -v / User/mojiway/stuff:/path/container (Mac)
```

In the assignment, we are making a postgres container with named volume:
```
docker container run -d --name psql -v psql:/var/lib/postgresql/data postgres:9.6.1
```

- the location of volume file has to be found at documentation:
[here](https://hub.docker.com/_/postgres/scans/library/postgres/9.6.12)

for running a container's log, we say:
```
dcoker container logs <container_name>
```

Also, if we want to have it run constantly we can add -f flag.
```
docker container logs -f <container_name>
```

Then we stop the container we just made:
```
docker container psql
```

Now, we want to make a new container with version 9.6.2 and see what happens.
```
docker container run -d --name psql2 -v psql:/var/lib/postgresql/data postgres:9.6.2
```
it automatically updates the second version and added very short updates to the older version.

In the assignment called bindmount-sample-1 we want to mount the folder to a container.

```
docker run -p 80:4000 -v $(pwd):/site bretfisher/jkyll-serve
```

