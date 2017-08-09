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
__1. Pull a Base Image:__
You can start off with either just base OS of your choice or OS+additional modules.
I'm starting off with ubuntu 16.04 as my [base image](https://hub.docker.com/_/ubuntu/). 
```
# docker pull <repository>:<tag>
docker pull ubuntu:16.04
```
Here, `ubuntu` is the name of the repository (official, public) and `16.04` is the tag/version.
You can check if the image is present locally by using `docker images` command.

__2. Start a Container:__
To log in the docker container, we can run the command below
```
docker run -i -t -net=host ubuntu:16.04 bash
```
where,
`-i` is running the image interactively.
`-t` is allocate a pseudo-TTY.
`-net=host` is to provide internet access to the container.

__3. Make Changes:__
Say, I install python 2.7
```
apt-get update
apt-get install python
```

__4. Commit Changes:__
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
Now when you type the command `docker images`, it will display the both these images.
If you'd like to remove the old image, use this command.
```
docker rmi <image_name>
docker rmi ubuntu:16.04
```

__5. Pushing changes to your docker hub account__
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
3. Build caffe and set environment variables.
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

__Example__: 
If you would like to do the same thing as above, but prefer to have a YAML (sort of) file that you can easily move around instead of an image (~3GB Compressed size), you could write your own docker file which is collection of commands you'd need to execute on top of a based image to build an image with your framework and dependency requirements. 

Below is an example of how to create a dockerfile.
 - First command should pull the base image from docker hub (ubuntu 16.04, cuda 8, cudnn 6)
 ```
 FROM nvidia/cuda:8.0-cudnn6-devel
 LABEL maintainer "Sakthivel Sivaraman <sakthi2793@gmail.com>"
 ```
 - Install dependencies for OpenCV 3.0
 ```
 RUN apt-get update
 RUN apt-get install --assume-yes build-essential cmake git \
    pkg-config unzip ffmpeg qtbase5-dev python-dev python-numpy \
    libopencv-dev libgtk-3-dev libdc1394-22 libdc1394-22-dev libjpeg-dev \
    libpng12-dev libtiff5-dev libjasper-dev libavcodec-dev libavformat-dev \
    libswscale-dev libxine2-dev libgstreamer0.10-dev libgstreamer-plugins-base0.10-dev \
    libv4l-dev libtbb-dev libfaac-dev libmp3lame-dev libopencore-amrnb-dev \
    libopencore-amrwb-dev libtheora-dev libvorbis-dev libxvidcore-dev v4l-utils \
    python-vtk liblapacke-dev libopenblas-dev checkinstall libgdal-dev && \
 ```
 - Build Opencv 3.0
 ```
 RUN git clone https://github.com/daveselinger/opencv.git /root/opencv && \
    cd /root/opencv && \
    git checkout 3.1.0-with-cuda8 && \
    mkdir build && cd build && \
    cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local -D WITH_CUDA=ON -D \
    WITH_CUBLAS=ON -D WITH_TBB=ON -D WITH_V4L=ON -D WITH_QT=ON -D WITH_OPENGL=ON -D \
    BUILD_PERF_TESTS=OFF -D BUILD_TESTS=OFF -DCUDA_NVCC_FLAGS="-D_FORCE_INLINES" .. && \
    make -j $(($(nproc) + 1)) && \
    sudo make install
 ```
 - Install dependencies for Caffe
 ```
 RUN apt-get update && apt-get install -y opencl-headers build-essential protobuf-compiler \
    libprotoc-dev libboost-all-dev libleveldb-dev hdf5-tools libhdf5-serial-dev \
    libopencv-core-dev  libopencv-highgui-dev libsnappy-dev \
    libatlas-base-dev cmake libstdc++6-4.8-dbg libgoogle-glog0v5 libgoogle-glog-dev \
    libgflags-dev liblmdb-dev git python-pip gfortran libopencv-dev && \
    pip install protobuf && pip install -U scikit-image && apt-get clean
 ```
 - Build Caffe
 ```
 # Clone repo
RUN git clone -b ${CAFFE_VERSION} https://github.com/BVLC/caffe.git /root/caffe && \
    cd /root/caffe

# Prepare Makefile.config for GPU support
RUN cp Makefile.config.example Makefile.config && \
    sed -i '/^# USE_CUDNN := 1/s/^# //' Makefile.config && \
    sed -i '/^# OPENCV_VERSION := 3/s/^# //' Makefile.config && \
    sed -i '/^# WITH_PYTHON_LAYER := 1/s/^# //' Makefile.config && \
    sed -i 's/\/usr\/local\/cuda/\/usr\/local\/cuda-8.0/g' Makefile.config && \
    sed -i 's/\/usr\/local\/include/\/usr\/local\/include \/usr\/include\/hdf5\/serial/g' Makefile.config && \
    sed -i 's/\/usr\/local\/lib/\/usr\/local\/lib \/usr\/lib\/hdf5\/x86_64-linux-gnu \/usr\/lib\/hdf5\/x86_64-linux-gnu\/hdf5\/serial/g' Makefile.config && \
    sed -i '/^PYTHON_INCLUDE/a    /usr/local/lib/python2.7/dist-packages/numpy/core/include/ \\' Makefile.config

RUN sudo ln -s /usr/lib/x86_64-linux-gnu/libhdf5_serial.so.10.1.0 /usr/lib/x86_64-linux-gnu/libhdf5.so && \
    sudo ln -s /usr/lib/x86_64-linux-gnu/libhdf5_serial_hl.so.10.0.2 /usr/lib/x86_64-linux-gnu/libhdf5_hl.so

RUN make -j 8 all py && \
    make -j 8 test && \
    make runtest
 ```
 - Set environment variables
 ```
 # Set up Caffe environment variables
ARG CAFFE_ROOT=/root/caffe
ARG PYCAFFE_ROOT=$CAFFE_ROOT/python
ARG PYTHONPATH=$PYCAFFE_ROOT:$PYTHONPATH \
ARG PATH $CAFFE_ROOT/build/tools:$PYCAFFE_ROOT:$PATH
RUN echo "$CAFFE_ROOT/build/lib" >> /etc/ld.so.conf.d/caffe.conf && ldconfig
 ```
 - Set working directory
 ```
 WORKDIR "/root/"
 CMD ["/bin/bash"]
 ```
 
To build this dockerfile, use this command.
```
docker build -f <dockerfile_name> .
```

*Note: You might easily find images or dockerfiles for most of the frameworks available readily. You don't always have to build it from scratch*

### Enabling GUI
The simple way is expose your xhost so that container can render to the correct display by reading and writing though the X11 unix socket.
```
$ xhost +
access control disabled, clients can connect from any host
```
Now, you can run the docker using this.
```
docker run -ti -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix <image_repo>:<tag>
```

### Saving and Loading Images
 After committing changes to an images, if you wish to save it locally as a tarball:
 ```
 $ docker save myImage > myImage.tar
 ```
 Now you can move this around(SCP, FTP etc.) and load it from any other host.
 ```
 $ docker load < myImage.tar
 ```
 
 ## Other Useful Commands
 Delete all containers
 ```
 docker rm $(docker ps -a -q)
 ```
 
 Delete all images
 ```
 docker rmi $(docker images -q)
 ```
 
 
 ## Issues:
 ### Docker login
 1. docker warning config.json permission denied
  ```
  sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
  sudo chmod g+rwx "/home/$USER/.docker" -R
  ```
2. Mounted host volume is not writable from container
Refer: [stackoverflow answer](https://stackoverflow.com/questions/34031397/running-docker-on-ubuntu-mounted-host-volume-is-not-writable-from-container) for options to solve the issue and understand how uid and gid can be matched between host and container. 

References:
  1. [A Practical Introduction to Docker Containers](https://developers.redhat.com/blog/2014/05/15/practical-introduction-to-docker-containers/)
  2. [A Beginner-Friendly Introduction to Containers, VMs and Docker](https://medium.freecodecamp.org/a-beginner-friendly-introduction-to-containers-vms-and-docker-79a9e3e119b)
  3. 
