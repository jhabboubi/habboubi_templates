
Table of Content

- [Prerequisite](#prerequisite)
- [gcloud](#gcloud)
- [jenkins](#jenkins)
- [vagrant tomcat8](#vagrant-tomcat8)
- [Dockerize Mariadb with auto restart and volume](#dockerize-mariadb-with-auto-restart-and-volume)
- [Dockerize MongoDB with auto restart and volume](#dockerize-mongodb-with-auto-restart-and-volume)
- [Vagrant provisioning and configuration with ansible](#vagrant-provisioning-and-configuration-with-ansible)
- [Dockerize SonarQube and Postgres for ARM64 Architecture (Raspberry Pi 4)](#dockerize-sonarqube-and-postgres-for-arm64-architecture-raspberry-pi-4)





## Prerequisite

  - Docker 


---

## gcloud
- ### Gcloud Docker image for ARM64 Architecture (Raspberry Pi 4)
  - built on [arm64v8/ubuntu](https://hub.docker.com/r/arm64v8/ubuntu)
  - Install GCloud && Updates

-   ### Usage
    - `docker build -t gcloud_arm64:latest .`
    - `docker run -it --rm --privileged --name gcloud gcloud_arm64`

---

## jenkins

- ### Jenkins Configured Docker Image for ARM64 (Raspberry Pi 4)
  - built on built on [jenkins4eval/jenkins](https://hub.docker.com/r/jenkins4eval/jenkins)
  - install Jenkins plugins using plugins.txt - deprecated and removed from Dockerfile
  - configure Jenkins using [JCasC](https://www.jenkins.io/projects/jcasc/)
  - [Documentation and Instructions](https://www.digitalocean.com/community/tutorials/how-to-automate-jenkins-setup-with-docker-and-jenkins-configuration-as-code)
  - JCASC refrences `server_ip:8080/configuration-as-code/reference`
  - [install jenkinsci plugin installation manager](https://github.com/jenkinsci/plugin-installation-manager-tool/)

- ### Usage
  Change the admin user and password
  - `docker build -t jenkins_arm64:latest . && docker run -d -u root --name jenkins -p 8080:8080 --env JENKINS_ADMIN_ID=admin --env JENKINS_ADMIN_PASSWORD=password --env JENKINS_IP=192.168.4.90:8080 -v /var/jenkins_home:/var/jenkins_home jenkins_arm64`
---
## vagrant tomcat8

- ### ubuntu 16.04 updated and tomcat 8 installed

- ### Usage
  - `vagrant up`
  - `vagrant destroy`
---
## Dockerize Mariadb with auto restart and volume
  - `docker network create app_name`
  - `docker run -p 3306:3306 --name db --network app_name -v data:/data --restart always -e MYSQL_DATABASE=test -e MYSQL_ROOT_PASSWORD=root -d mariadb`
---
## Dockerize MongoDB with auto restart and volume
  -  `docker run -d --name mongodb -p 27017:27017 -e MONGO_INITDB_ROOT_USERNAME=root -e MONGO_INITDB_ROOT_PASSWORD=root -e MONGO_INITDB_DATABASE=test --network app_name -v mangodb:/data --restart always mongo`
---
## Vagrant provisioning and configuration with ansible
  - make sure virtualbox is installed on the host
  - make sure vagrant installed on host
  - use the terminal and change directory to vagrant_and_ansible
  - `vagrant up` for all vms or `vagrant up m1 m2 m3` 
  	- m1: ubuntu 18.04 ansible control and docker is installed
	- m2: ubuntu 18.04 
	- m3: ubuntu 18.04
	- m4: centos 8
	- m5: fedora 33

---

## Dockerize SonarQube and Postgres for ARM64 Architecture (Raspberry Pi 4)
- create network `docker create network sonarqube_network`
- create postgres container `docker run --name postgres -d --restart unless-stopped -p 5432:5432 -e POSTGRES_PASSWORD=root -v postgres_data:/var/lib/postgresql/data postgres:12.2`
- create database in postgres `docker exec -it postgres bash` then `psql -U postgres` then create schema `create database sonar;`
- create sonarqube container `docker run -u sonarqube -d --name sonarqube \
           -v sonarqube_data:/opt/sonarqube/data \
           -v sonarqube_extensions:/opt/sonarqube/extensions \
           -v sonarqube_logs:/opt/sonarqube/logs \
           -e SONARQUBE_JDBC_USERNAME="postgres" \
           -e SONARQUBE_JDBC_PASSWORD="root" \
           -e SONARQUBE_JDBC_URL="jdbc:postgresql://postgres:5432/sonar" \
           --network sonarqube_network \
           -p 9000:9000 \
           koolwithk/sonarqube-arm:9.2.4-community`

- **Quick Fix**
  - if the container crashes... if you see this error in /opt/sonar/logs/es.log :
    or `docker logs sonarqube`

    2020.05.17 14:46:12 ERROR es[][o.e.b.Bootstrap] node validation exception
    [1] bootstrap checks failed
    [1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
    Execute this on your node :

    `sudo sysctl -w vm.max_map_count=262144`