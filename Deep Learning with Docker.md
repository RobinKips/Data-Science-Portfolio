
## Create your own Deep Learning development environment using Docker

Create up your own DL environement based on docker containers 

## Index 

- 1. Introduction 
	* Why Deep Learning ? 
	* Why Docker ?
   	* Prerequisite

- 2. Install Docker 
	* What is Docker ? 
	* Docker installation
	* Shared volume
	* Run your container
- 3. Install Jupyterhub
	* What is Jupyterhub ? 
	* Jupyterhub installation 
	* Jupyterhub configuration
- 4 : Installing a Keras kernel for Jupyter
	* Virtual environments with Conda
	* installing Keras 
	* Using Keras in Jupyter 
- 5: Bonus : docker tips for improving your container
	* Create your Docker image
	* Share a host directory with your container 
	* run your container as a service 

---------------------------------------

## 1. Introduction 

### Why Deep Learning ? 

Appart from being surrounder by a lot of hype, **Deep Learning** (DL) is a powerfull set of methods that will help you adress many complex problematic. One of the most famous and groundbreaking application of Deep Learning is related to the field of **Computer Vision** and image classification problems. 

Classifying an image is a difficult task for computers, that have to face many complex issues : viewpoint variation, illumination conditions, etc. Deep Learning techniques have proven particularly powerfull for adressing such issues. 

![alt text](http://cs231n.github.io/assets/challenges.jpeg "computer vision challenging issues" )

This tutorial will help you to get started with Deep Learning by creating your own DL environment from scratch. 

### Why Docker ?

One important difficulty that comes with Deep Learning is the environment setup. There is a variety of DL oriented libraries, but they all require very specific dependances which makes them difficult to plug to your already existing development environment. That is where **Docker** will be of great help. 

We will use the docker container technology in order to easily build an isolated Deep Learning platform, and preserve your current environment from any regression. 

In addition, Docker is a particularly trending technology that is worth trying, as it will help you with any kind of application development and deployment. 


### Prerequisite  

All you need to get started is a Linux machine. If you do not have any, I encourage you to launch a cloud instance with **Amazon Web Service** (AWS) . Even though optimal Deep Learning environment are based on GPU architectures, using a free AWS machine is enough to get you started and adress problems of modes complexity. 

This tutorial is designed for being used with **Red Hat Entreprise Linux** or **CentOS** distributions. 


## 2. Install Docker 
### What is Docker ? 

![alt text](https://images.mondedie.fr/s9ZCIrQ4/ErDsRisu.png "Docker Architecture illustration")


### Docker installation
```
sudo yum update -y
#install docker 
sudo yum install -y docker
#start docker service
sudo service docker start
#add ec2 user to the docker group to avoir using sudo 
sudo usermod -a -G docker ec2-user
```

### Docker installation

When adressing Deep Learning problems you will have to deal with potentially large datasets. In order to avoid duplicating your data in your container, you can use a share volume to easily read host data from any of your running container.  
Thus, create a new directory that will be mounted as a shared volume : 
```
mkdir /home/share_docker
chmod 555 /home/share_docker
```

### Run your container

You are now ready to run your container. The `docker run` commands allows to create and run a container : 

```
docker run --name DL_platform -dti -p 443:443 -v /home/share_docker:/home/share_docker:z centos 
```

This needs a few comments. It is important that you understand all the argument passed to the previous command. They will help you understand all the potential of docker and adapt it to your own problems.
* `--name`  :sets the name of your container
* `--dti` : the -d  option is used for running the container  in the background. Using -ti allows to use an interactive terminal with your container. 
* `-p`  : used fro  to map a port of the host to a container port. 
* `-v` : used for sharing volumes between host and container. Notice the `:z` options that allows to pass SE linux tags from the host, solving many issues. 
* `centos` : the last argument is the image used for creating a container. Here we run an container from a raw centos image. We will see that you can create your own docker images. 


Your containter is now up and runing. You can have a look at all your containers with the command `docker ps -a`. If for some reason your container is not running anymore, you can start it with the command `docker start DL_platform`.

You can now start a terminal in your container with the command : 

```
docker exec -it DL_platform bash
```
	
## 3. Install Jupyterhub





### Anaconda installation 

Install several compenents for later needs : 

```
yum update -y  
yum  install -y wget bzip2
```

Install Anaconda. Contains Python 3, advanced python environment manager, and all the most popular data science libraries. 

```
#download Anaconda
wget --quiet http://repo.continuum.io/archive/Anaconda3-4.1.1-Linux-x86_64.sh -O /tmp/Anaconda3-4.1.1-Linux-x86_64.sh 
#make the script executable 
chmod +x /tmp/Anaconda3-4.1.1-Linux-x86_64.sh
#installing to /opt/anaconda
/tmp/Anaconda3-4.1.1-Linux-x86_64.sh  -b -p /opt/anaconda
#cleanup
rm -f /tmp/*
```

Add anaconda binaries repository to yhour path. All the user that want to use anaconda must do so. 
```
echo 'PATH=$PATH:/opt/anaconda/bin' >> ~/.bashrc
```

### Jupyterhub installation 
#### prerequisite installations

First install **nodejs/npm** (Node Package Manager). For RHEL, Node.js is available from the NodeSource Enterprise Linux and Fedora binary distributions repository. 
```
curl --silent --location https://rpm.nodesource.com/setup_4.x | bash -
yum install -y nodejs
```
Then, install the **configurable http proxy** used by Jupyterhub : 
```
npm install -g configurable-http-proxy
```

### Installing Jupyterhub  : 

Install jupyterhub 
```
/opt/anaconda/bin/pip install jupyterhub
```

Then we must upgrade the jupyter notebook. There is a small workarround here in order to avoid a bug in upgrading python setuptools using anaconda pip. Using the `--ignore-installed` argument we ignore the installed package  and force the  reinstalling instead. 
```
opt/anaconda/bin/pip install --upgrade --ignore-installed  setuptools
```

You can now safely upgrade the notebook : 
```
opt/anaconda/bin/pip install --upgrade notebook
```

Setup an alias for jupyterhub binaries . Add the following line to ` /root/.bashrc`

```
echo 'alias jupyterhub="/opt/anaconda/bin/jupyterhub"' >> ~/.bashrc 
```

### 2.2. Configuring Jupyterhub  : 


Generate default configuration file for Jupyterhub in `/etc/jupyterhub/'

```
mkdir /etc/jupyterhub 
cd /etc/jupyterhub 
jupyterhub --generate-config

yum install -y  openssl
#autisigned certificate
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/my_key.pem -out /etc/ssl/my_cert.pem
```

You can start the jupyterhub server with the following command :  

```
jupyterhub --port 443 --ssl-key /etc/ssl/my_key.pem --ssl-cert /etc/ssl/my_cert.pem
```
the jupyterhub server should now start to listen on the port 443 of your container. Check that this port is indeed binded to the host. 

## Installing keras kernel : 

```
/opt/anaconda/bin/conda create --name keras_kernel python=3.5.2  -y 
source /opt/anaconda/bin/activate keras_kernel 
/opt/anaconda/bin/conda install  -y theano=0.8.2
/opt/anaconda/bin/conda install -y h5py=2.6.0 
pip install keras==1.1.1 
/opt/anaconda/bin/conda install -y scikit-learn=0.18.1 pandas=0.19.1 ipykernel=4.5.1 matplotlib=1.5.3 pillow=3.4.1
source /opt/anaconda/bin/deactivate 
```
### Build a Jupyter docker image : 

Now that we corrrectly installed jupyterhub in a containner we can save it as a docker image. Thus, we will be able to use this image to start a new container running jupyterhub. We save the container jupyter_deploy as the image jupyter_img_v1.  

commit jupyter_deploy jupyterhub_img_v1 




## 3. Running the Jupyterhub container  : 

### Open a port on the host 

For deploying the jupyterhub, **bind a container port to an host port**. First, we need to open a port on the host machine that runs the docker deamon. Here we open port 8081 : 

```
firewall-cmd --permanent --zone=public --add-port=3389/tcp
firewall-cmd --reload
```

Then, create a share docker volume on the host to ensure data persistance. When using the `docker volume create`command, we can attribute a name to a docker volume for easy volume management. However the data will be on the host by default at : `var/lib/docker/volumes/<volumename>/_data`. It is not currently possible to specify path, even though the workarround `docker volume create --opt type=none --opt device=<host path> --opt o=bind` could work. Instead, we choose to manually mount a path on the Docker host’s filesystem as a volume inside the container. 

### Run a new  Jupyterhub container : 

Use the following command to run the jupyterhub container : 
```
docker run -tid --name jupyter_deploy -p 8081:8000 -v /home/share_init:/home/share_init:z jupyterhub_img_v1 
```

The port 8081 of the host is binded to the port 8000 of the container, the default port for jupyterhub. When writing data in `/home/share_init` from Jupyter, the data will be written in the host to `/home/share_init`. Using the same volume option with another container allows to access the same shared folder. 

Finally, manually start the jupyterhub server connecting to the docker : 

```
#from the host : 
docker exec -it jupyter_deploy bash 

#from the container : 
cd /srv
jupyterhub --no-ssl
```
The service is started from `/srv` since the files `jupyterhub.sqlite` and `jupyterhub_cookie_secret` are created in the folder from where jupyterhub is started. Deleting these files will result in disconecting the users. 

### Create the users manually 

Jupyterhub allows to run a multi-user notebook server. Thus, users must be created from the os hosting jupyterhub. You can  create users manually using `useradd user_1` and then set the corresponding pasword with `passwd user_1`. 



### Jupyterhub configuration
- 4 : Installing a Keras kernel for Jupyter
	* Virtual environments with Conda
	* installing Keras 
	* Using Keras in Jupyter 
- 5: Bonus : docker tips for improving your container
	* Create your Docker image
	* Share a host directory with your container 
	* run your container as a service 



```

FROM centos

MAINTAINER Robin KIPS <robin.kips@gmail.com>

# install Python with Anaconda
RUN yum -y install wget-1.14 bzip2-1.0.6 gcc-c++-4.8.5  && \
	wget --quiet http://repo.continuum.io/archive/Anaconda3-4.1.1-Linux-x86_64.sh -O /tmp/Anaconda3-4.1.1-Linux-x86_64.sh && \
	chmod +x /tmp/Anaconda3-4.1.1-Linux-x86_64.sh && \
	/tmp/Anaconda3-4.1.1-Linux-x86_64.sh  -b -p /opt/anaconda && \
	rm -f /tmp/* 

	# install npm, nodejs & js dependencies
RUN curl --silent --location https://rpm.nodesource.com/setup_4.x | bash - && \
	yum install -y nodejs-4.6.0 && \
	npm install -g configurable-http-proxy@1.3.0 && \ 
	opt/anaconda/bin/pip install jupyterhub==0.6.1  && \ 
	opt/anaconda/bin/pip install --upgrade --ignore-installed  setuptools==28.3.0 && \ 
	opt/anaconda/bin/pip install --upgrade notebook==4.2.3 && \
	yum install -y  openssl && \
	openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/my_key.pem -out /etc/ssl/my_cert.pem
	
# install keras kernel
RUN /opt/anaconda/bin/conda create --name keras_kernel python=3.5.2  -y && \
	source /opt/anaconda/bin/activate keras_kernel && \
	/opt/anaconda/bin/conda install  -y theano=0.8.2 && \
	/opt/anaconda/bin/conda install -y h5py=2.6.0 && \
	pip install keras==1.1.1 && \
	pip install flask==0.11.1 && \
	/opt/anaconda/bin/conda install -y scikit-learn=0.18.1 pandas=0.19.1 ipykernel=4.5.1&& matplotlib=1.5.3 pillow=3.4.1 && \
	source /opt/anaconda/bin/deactivate 

#set alias : 
RUN echo 'alias conda="/opt/anaconda/bin/conda"' >> ~/.bashrc && \
	echo 'alias activate="/opt/anaconda/bin/activate"' >> ~/.bashrc && \
	echo 'alias deactivate="/opt/anaconda/bin/deactivate"' >> ~/.bashrc && \
	echo 'alias python_keras="/opt/anaconda/envs/keras_kernel/bin/python3"' >> ~/.bashrc && \
	echo 'alias jupyterhub="/opt/anaconda/bin/jupyterhub"' >> ~/.bashrc 

#set ENV
ENV KERAS_BACKEND=theano
ENV PATH=/opt/anaconda/bin:$PATH

#spawn jupyter in /home/ dir for access shared directories.
RUN mkdir -p /srv/jupyterhub/ && \
    cd /srv/jupyterhub/ && \ 
    /opt/anaconda/bin/jupyterhub --generate-config && \
    echo "c.Spawner.notebook_dir = '/home/'" >> jupyterhub_config.py

WORKDIR /srv/jupyterhub/
EXPOSE 8000
 
#start jupyterhub 
CMD ["jupyterhub","--port 443", "--ssl-key", "/etc/ssl/my_key.pem", "--ssl-cert", "/etc/ssl/my_cert.pem", "-f", "jupyterhub_config.py"]
```
