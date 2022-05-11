# cro829
IBM Resilency Orchestration 8.2.9 container

[![License](https://img.shields.io/badge/License-Apache2-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0) [![Slack](https://img.shields.io/badge/Join-Slack-blue)](https://callforcode.org/slack) [![Website](https://img.shields.io/badge/View-Website-blue)](https://code-and-response.github.io/Project-Sample/)

*Read this in other languages: [English](README.md), [Español](README.es_co.md).*

## Introduction to Resilency Orchestration 8.2.9

[![Watch the video](https://higherlogicdownload.s3.amazonaws.com/IMWUC/DeveloperWorksImages_blog-storageneers/ScreenShot2018-10-03at4.32.30PM.png)](https://youtu.be/YRKI2deypuI)


## Prepare CRO training environment:

### Prepare CRO environment with docker/podman

Prepare a Virtual Machine with RHEL 8.x (Linux Red Hat Enterprise) or Ubuntu with Docker
    1. On Apple Mac use docker for Mac and command line
    2. On windows use docker for windows and select linux containers enablinbg hyper-v virtualization. It is easier to use VirtualBox and start Ubuntu VM with docker
    3. On RHEL 8.x podman is installed by default and is fully command compatible with docker commands.  For scenarios where need docker-compose you can use podman-compose

### Pre-requisite preparation

    1. Import and run RHEL 8.4 UBI (minimal) base image
        $ docker run -it --name cro -h crort -p 8080:8080 registry.access.redhat.com/ubi8/ubi:8.4 bash
        
    2. Install library
       [root@crort /]# dnf -y install hostname libXtst net-tools iputils procps-ng rsync sudo unzip wget
       
    3. Download Tomcat 9.0.54 and install on /opt
       [root@crort /]# wget -qO- https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.54/bin/apache-tomcat-9.0.54.tar.gz | tar -xzf - -C /opt
       
    4. Download MariaDB 10.5.9 installation rpm's
       [root@crort /]# wget https://downloads.mariadb.com/MariaDB/mariadb-10.5.9/yum/rhel8-amd64/rpms/MariaDB-shared-10.5.9-1.el8.x86_64.rpm
       [root@crort /]# wget https://downloads.mariadb.com/MariaDB/mariadb-10.5.9/yum/rhel8-amd64/rpms/MariaDB-common-10.5.9-1.el8.x86_64.rpm
       [root@crort /]# wget https://downloads.mariadb.com/MariaDB/mariadb-10.5.9/yum/rhel8-amd64/rpms/MariaDB-server-10.5.9-1.el8.x86_64.rpm
       [root@crort /]# wget https://downloads.mariadb.com/MariaDB/mariadb-10.5.9/yum/rhel8-amd64/rpms/MariaDB-client-10.5.9-1.el8.x86_64.rpm
       [root@crort /]# wget https://downloads.mariadb.com/MariaDB/mariadb-10.5.9/yum/rhel8-amd64/rpms/galera-4-26.4.7-1.el8.x86_64.rpm

    5. Download boost-program-options (required for galera)
       [root@crort /]# wget http://mirror.centos.org/centos/8/AppStream/x86_64/os/Packages/boost-program-options-1.66.0-10.el8.x86_64.rpm

    6. Install MariaDB, start container and change root password
       [root@crort /]# rpm -Uvh boost-program-options-1.66.0-10.el8.x86_64.rpm
       [root@crort /]# dnf --disableplugin=subscription-manager install -y MariaDB-server galera MariaDB-client MariaDB-shared MariaDB-backup MariaDB-common
       [root@crort /]# /etc/init.d/mysql start
       [root@crort /]# mysqladmin -f -u root password 'Passw0rd'

### Resiliency Orchestration Install

    1. On other linux shell (Outside container), copy product installation binaries (Server.tar.gz) inside the container:
       $ docker copy Server.tar.gz crort:/tmp
       
    2. Execute RO installation steps starting on page 77 in console mode:
       [root@crort /]# cd tmp
       [root@crort /]# tar xvzf Server.tar.gz
       [root@crort /]# sed 's/LOCAL_HOST_SERVER=/LOCAL_HOST_SERVER=10.0.0.2/
       > s/DATABASE_PASSWORD=/DATABASE_PASSWORD=Passw0rd/
       > s/LICENSE_ACCEPTED=/LICENSE_ACCEPTED=TRUE/
       > s/SUPPORT_USER_PASSWORD=/SUPPORT_USER_PASSWORD=Orchestration123./
       > s/SANOVI_USER_PASSWORD=/SANOVI_USER_PASSWORD=Orchestration123./
       > s/MODIFY_SYSTEM_FILES=0/MODIFY_SYSTEM_FILES=1/
       > s/TOMCAT_HOME=.*/TOMCAT_HOME=\/opt\/apache-tomcat-9.0.37/' /tmp/PanacesServerInstaller.properties > /tmp/PanacesServerCro.properties
       [root@crort /]# /tmp/install.bin -f /tmp/PanacesServerCro.properties
    3. Ejecute los pasos post instalación descritos en la guia de instalación de CRO a partir de la página 85 (numeral 5.6) para el modo consola

### Start RO services

    1. Start RO services in container:
       export EAMSROOT=/opt/panaces
       /etc/init.d/mysql status || /etc/init.d/mysql start
       cd /opt/panaces/bin
       ./panaces restart
       
    2. On other linux shell (Outside container), save running container image to persist installation (containers are not persistent by nature)
       $ docker commit crort cro:8.2.9
    
And that's up! already have a base image of RO we can use for training or test
    $ docker images

REPOSITORY                           TAG             IMAGE ID      CREATED        SIZE
localhost/cro                        8.2.9         05fc28b5ad50  2 minutes ago   4.15 GB
