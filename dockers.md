# Dockers for Deep Learning

## What are Dockers?
*Docker is the world’s leading software container platform. Developers use Docker to eliminate “works on my machine” problems when collaborating on code with co-workers.*
Unlike VMs which emulate virtual hardware, Docker uses containers, which share the operating system of the host machine, but still keep your applications and software isolated from your host environment. 
This reduces a lot of storage overhead that comes along with a VM.

One of the key advantages of using Docker is it’s centralized image management server, called a Registry Server. 
The Docker project maintains a public registry server which hosts images they maintain, as well as images created by the community. 
This registry service is free to use, as long as the images are public.

A good resource to know more about Docker and Containerization technology as a whole read [this blog](https://becominghuman.ai/docker-for-data-science-part-1-dd41e5ef1d80).
 
## Docker Basics
### Docker Installation
(Ubuntu) This is pretty straightforward. You can either follow the official [Docker Documentation](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/) or the [DigitalOcean blog](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-16-04)

### Working without sudo
The Docker daemon runs as root, which means that users will need to use sudo to run Docker commands.
To avoid having to use sudo for every Docker command, simply add your user(s) to the docker group with this command.
```
usermod -aG docker <username>
```

### Basic steps to understand Docker usage
1. Pull a Base Image:
You can start off with either just base OS of your choice or OS+additional modules.
I'm starting off with ubuntu 16.04 as my [base image](https://hub.docker.com/_/ubuntu/). 
```
# docker pull <repository>:<tag>
docker pull ubuntu:16.04
```
Here, `ubuntu` is the name of the repository (official, public) and `16.04` is the tag/version.
You can check if the image is present locally by using `docker images` command.

2. Start a Container:
To log in the docker container, we can run the command below
```
docker run -i -t -net=host ubuntu:16.04 bash
```
where,
`-i` is running the image interactively.
`-t` is allocate a pseudo-TTY.
`-net=host` is to provide internet access to the container.

3. Make Changes:
Say, I install python 2.7
```
apt-get update
apt-get install python
```

4. Commit Changes:
Any changes made to the docker file lasts only as long as the container is running. So to save changes, you need to commit them. 
Create a new image from a container’s changes.
First, find the container ID. Open a new terminal and type `docker ps` - lists all the running containers. 
```
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
fd08a7fee149        ubuntu:16.04        "bash"              8 minutes ago       Up 8 minutes                            nostalgic_thompson
```
To commit changes, type the following command:
```
# docker commit -m <commit_message> <container_ID> <repo:tag>
docker commit -m "first commit" fd08a7fee149 ubuntu:16.04
```
If you'd like to save it as a different image:
```
docker commit -m "first commit" fd08a7fee149 myProject:v1
```
Now when you type the command `sudo docker images`
If you'd like to remove the old image
```
docker rmi <image_name>
docker rmi ubuntu:16.04
```

5. Pushing changes to your docker hub account
First, you need to create a docker hub account.
Then, to login locally, type the following command and enter your credentials 
```
docker login
```
Now, you can push the new image to your repo by,
```
# docker push <dockerhub_username>/<repo>:<tag>
docker push sakthi2793/myProject:v1
```
This should upload all the image layers to your account. 


## Using Dockers for Deep Learning
Along with docker, you also need to install nvidia-docker.
To access GPU, when you run the container, use `nvidia-docker` instead of `docker`. 
To verify if you have GPU access, use command `nvidia-smi`. 

### Method 1
1. Pull Base image
2. Install necessary packages and setup the desired Deep learning framework (don't forget to commit)
3. Experiment with Deep learning models using Data Volumes (to load/save data from/to local host)

__Example__: 
Say you want to set up a custom caffe framework with GPU support. 
1. Download base image: 
```
docker pull nvidia/cuda:8.0-cudnn5-runtime-ubuntu16.04
```
2. Install required [dependencies for caffe]. (https://github.com/sakthi-s/Installation-Instructions/blob/master/caffe.md)
3. Build caffe. 
4. Commit changes as a new image. 
5. Run container using data volumes:
```
# `-v` <local host directory>:<mount location in container>
docker run -it --net=host -v /home/sakthi/data/:/data/ -v /home/sakthi/models/:/models/ sakthi2793/caffe-mod:v1
```
Here `-v` is used to mount volumes. This saves me from creating an image of size 100GB+ (size of the data). `sakthi2793` is my dockerhub username and `caffe-mod:v1` is the repo:tag name.
Similarly, `-p` can be use for port map from host to container. Eg. `-p 8080:80` port 8080 on host is connected to port 80 on container.

### Method 2
1. Create Dockerfile
2. Build Docker file
3. Experiment with Deep learning models using Data Volumes (to load/save data from/to local host)

*Note: You might easily find images or dockerfiles for most of the frameworks available readily. You don't always have to build it from scratch*

### Enabling GUI

### Saving and Loading Images
 After committing changes to an images, if you wish to save it locally as a tarball:
 ```
 $ docker save myImage > myImage.tar
 ```
 Now you can move this around(SCP, FTP etc.) and load it from any other host.
 ```
 $ docker load < myImage.tar
 ```
 
 ## Other Options
 
 ## Issues:
 ### Docker login
 1. docker warning config.json permission denied
  ```
  sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
  sudo chmod g+rwx "/home/$USER/.docker" -R
  ```

References:
  1. [A Practical Introduction to Docker Containers](https://developers.redhat.com/blog/2014/05/15/practical-introduction-to-docker-containers/)
  2. [A Beginner-Friendly Introduction to Containers, VMs and Docker](https://medium.freecodecamp.org/a-beginner-friendly-introduction-to-containers-vms-and-docker-79a9e3e119b)
  3. 
