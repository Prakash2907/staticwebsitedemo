# Tutorial: How to deploy a static website to DigitalOcean on a Managed Kubernetes Cluster


This guide describes how to deploy static website on digital ocean kubernetes managed cluster cluster. The guide provides step by step tutorial to Containerize the static website and then deploy it to a managed kubernetes cluster. The repository contains the code, the deployment files required to setup static website on managed kubernetes cluster.

# Prerequisite
 - Digital Ocean Account. please signup if you don't have digital ocean account [here](https://cloud.digitalocean.com/registrations/new)
 - A ubuntu virtual machine with docker installed and running on digital ocean platform. [here](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)
 - Dockerhub id to upload/download image. [here](https://docs.docker.com/accounts/create-account/)
 - Git Client - To clone the repository. (comes pre-installed on ubuntu) 
 - Docker file and index.html required to create static website. [website-files](https://github.com/Prakash2907/staticwebsitedemo/tree/main/Static)


# Step 01 - Containerizing the static website with docker

 - Make sure docker is installed in the system & running
   ```shell
   root@ubuntu-s-1vcpu-1gb-blr1-01:~# docker version
   ```
 - Clone this repository locally
   ```shell
   git clone https://github.com/Prakash2907/staticwebsitedemo.git
   ```
- Now go to the static website folder and check for index.html and Dockerfile. index.html will be used for static website and Dockerfile will be used for creating docker image.
  ```shell
  root@ubuntu-s-1vcpu-1gb-blr1-01:~# cd staticwebsitedemo/
  root@ubuntu-s-1vcpu-1gb-blr1-01:~/staticwebsitedemo# cd Static/
  root@ubuntu-s-1vcpu-1gb-blr1-01:~/staticwebsitedemo/Static# ls
  Dockerfile  index.html
  root@ubuntu-s-1vcpu-1gb-blr1-01:~/staticwebsitedemo/Static#
  ```
- To containerize the website you need to create docker image from the Dockerfile. The docker file has below 2 lines:
  ```shell
  FROM nginx:stable-alpine
  COPY index.html /usr/share/nginx/html
  ```
  The FROM command will is used to specify the base image which needs to be used ti create docker image.


