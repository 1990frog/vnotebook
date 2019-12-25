[TOC]

# Docker Benefits
The big benefits of packaging an application along with its server configuration using a Dockerfile are:

+ You don't forget how your server was configured. The Dockerfile remembers that for you.
+ You can easily run your application on a new Docker host. Just deploy the Docker image for your application to that Docker host, and start it up. It's all automated.
+ Cluster tools like Kubernetes and Docker Swarm can easily manage Docker containers in clusters for you.
+ Many cloud platforms can deploy Docker containers easily. Docker is thus a simple way to become a bit more cloud independent.
+ Docker containers are an easy way for your clients to install your application on their own servers.
# What is a Docker Container?
The Linux operating system has several features which allows for containerization of applications running on top of the operating system (OS). These containerization features provide the ability to separate the file system and networks of containerized applications. In other words, one containerized application cannot access the file system or network of another containerized application, unless you explicitly allow it. Docker uses these Linux containerization features and exposes them via an easy to use set of tools.

![docker-introduction-1](_v_images/20191225145925696_924508039.png)
# Docker Containers vs. Virtual Machines
Docker containers are similar in nature to virtual machines. However, a virtual machine has an extra OS in the total stack. A virtual machine has a VM OS, and then the VM is running on some computer which also has its own OS.

A Docker container, on the other hand, does not have its own internal OS. The container runs directly inside the host Linux OS. Thus, a Docker container is smaller in size, since it does not contain the whole VM OS. Docker containers can also perform better, as there is no virtualization of the VM necessary.

![docker-introduction-2](_v_images/20191225153802925_1656873424.png)


