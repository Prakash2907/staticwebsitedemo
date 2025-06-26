# Tutorial: How to deploy a static website to DigitalOcean on a Managed Kubernetes Cluster


This guide describes how to deploy static website on digital ocean kubernetes managed cluster cluster. The guide provides step by step tutorial to Containerize the static website and then deploy it to a managed kubernetes cluster. The repository contains the code, the deployment files required to setup static website on managed kubernetes cluster.

# Prerequisite
 - Digital Ocean Account. please signup if you don't have digital ocean account [here](https://cloud.digitalocean.com/registrations/new)
 - A ubuntu virtual machine with docker installed and running on digital ocean platform. [here](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)
 - Dockerhub id to upload/download image. [here](https://docs.docker.com/accounts/create-account/)
 - Git Client - To clone the repository. (comes pre-installed on ubuntu) 
 - Docker file and index.html required to create static website. [website-files](https://github.com/Prakash2907/staticwebsitedemo/tree/main/Static)


# Step 01 - Containerizing the static website with docker

 - Launch a virtual machine with ubuntu os in your digital ocean account. We will use this machine to build the docker image. You can use minimum size VM.
 - Make sure docker is installed in the system & running. Please refer to the pre-reqisite section on installing docker on ubuntu.
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
  ```
- To containerize the website you need to create docker image from the Dockerfile. The docker file has below 2 lines:
  ```shell
  FROM nginx:stable-alpine
  COPY index.html /usr/share/nginx/html
  ```
  The FROM command will is used to specify the base image which needs to be used to create docker image. In this case we are using nginx:stable-alpine image as it   doesn't have any critical vulnerabilities. The best practice says use a secure image always. The tag specify which image version to use. If you don't specify      the tag it will default to the latest version which may or maynot work for your application.

  The COPY command copies the build context (in our case index.html) to the required directory. It is better to keep the Dockerfile and the build context in the     single directory.

 - Building the docker image. The image is build using docker build command which is run in the same directory as the docker file as we saw earlier.
   ```shell
   root@ubuntu-s-1vcpu-1gb-blr1-01:~/staticwebsitedemo/Static# docker build -t prakashjha1986/k8-demo .         
   ```
   The newly built image is currently stored locally. You can check it by using "docker image ls"
   ```shell
   root@ubuntu-s-1vcpu-1gb-blr1-01:~/staticwebsitedemo/Static# docker image ls
   REPOSITORY               TAG       IMAGE ID       CREATED         SIZE
   prakashjha1986/k8-demo   latest    7486083978bf   7 minutes ago   48.2MB
   ```
   We need to push the image to docker hub. For this we will 1st login to the docker hub using username and pasword and use the below command.
   ```shell
   root@ubuntu-s-1vcpu-1gb-blr1-01:~/staticwebsitedemo/Static# docker login -u prakashjha1986 
   Login Succeeded
   root@ubuntu-s-1vcpu-1gb-blr1-01:~/staticwebsitedemo/Static# docker push prakashjha1986/k8-demo
   Using default tag: latest
   The push refers to repository [docker.io/prakashjha1986/k8-demo]
   7e38ca2111b1: Pushed
   ```
   You can see the below screenshot of the image now upload to our repository on dockerhub
   ![DockerImage](https://github.com/user-attachments/assets/982f515a-2618-49df-a0b1-14f465a85f0e)
   
# Step 02 - Creating a managed kubernetes cluster on Digital Ocean

 - Login to DigitalOcean [here](https://cloud.digitalocean.com/login)

 - Click Create Button on top right and select Kubernetes.
   ![KubernetesClusterCreate](https://github.com/user-attachments/assets/2f315c1c-52dc-415b-829d-54f10e3420a4)

 - Select the latest version as a best practice recommended by Digital Ocean and select the region where you need to lauch cluster
   ![ClusterCreate2](https://github.com/user-attachments/assets/7d5d8186-2efe-40d7-b306-8e2291ea18a0)
   
 - Keep the VPC setting as it is.
   
 - In the Cluster Capacity section, Provide a Node pool name and change number of Nodes to 2. Check "Set Node Pool to Autoscale" which will increase the number on    Nodes, based on CPU utilization.
   ![Clustercapacity](https://github.com/user-attachments/assets/6cfdcfd0-9505-4e79-be23-3efdc06d8f40)
   
 - There is also a high avaialbility option for controlplane, which will run control plane components in HA mode. This can be used in the production         environment where high availability is very essential.
   
 - Lastly, add the cluster name under finalize section and then create the cluster.You will see your cluster as shown below.

   ![ClusterCreate3](https://github.com/user-attachments/assets/74cc8775-bbde-4313-878f-5753a825ed0a)
   
# Step 03 - Connecting to kubernetes cluster on Digital Ocean using doctl and kubectl
 - Install doctl as per the steps given in the official document [here](https://docs.digitalocean.com/reference/doctl/how-to/install/)
 - To provide doctl access to the digital ocean environment create an API token & click on generate the token [API token  generation](https://cloud.digitalocean.com/account/api/tokens). It will generate a token which you will use to authenticate doctl as per the given below steps.
   ![APIToken](https://github.com/user-attachments/assets/81e2b756-2fd8-4d73-878f-e3306ea74e8e)
   ```shell
   $ doctl auth init
   Please authenticate doctl for use with your DigitalOcean account. You can generate a token in the control panel at             https://cloud.digitalocean.com/account/api/tokens

   ❯ Enter your access token:  ●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●

   Validating token... ✔      
   ```
 - Once the doctl is authenticated install kubectl as per the instruction given [here](https://kubernetes.io/docs/tasks/tools/)
 - now run the below commnd which will create a .kube/config file to authenticate kubernetes
    ```shell
    $ doctl kubernetes cluster kubeconfig save cbd4fc7a-a5f6-45e3-861f-389bf992ca59
    Notice: Adding cluster credentials to kubeconfig file
    Notice: Setting current-context to do-nyc1-mystaticwebsite
    ```
  - Run "kubectl get nodes" command to access the nodes of the kubernetes cluster
    ```shell
    $ kubectl get nodes
    NAME            STATUS   ROLES    AGE   VERSION
    webpool-l3ta2   Ready    <none>   66m   v1.33.1
    webpool-l3tap   Ready    <none>   65m   v1.33.1
    ```





 - 
