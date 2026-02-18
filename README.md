exited(0) -> nothing wrong
exited (1) -> error occured in the container

podman-run -> to look into every arguemnt


podman run -it ubi8-micro
sh-4.4# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
sh-4.4# cat /ect/os-release
cat: /ect/os-release: No such file or directory
sh-4.4# cat /etc/os-relese
cat: /etc/os-relese: No such file or directory
sh-4.4# cat /etc/os-release
NAME="Red Hat Enterprise Linux"
VERSION="8.10 (Ootpa)"
ID="rhel"
ID_LIKE="fedora"
VERSION_ID="8.10"
PLATFORM_ID="platform:el8"
PRETTY_NAME="Red Hat Enterprise Linux 8.10 (Ootpa)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:redhat:enterprise_linux:8::baseos"
HOME_URL="https://www.redhat.com/"
DOCUMENTATION_URL="https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8"
BUG_REPORT_URL="https://issues.redhat.com/"

REDHAT_BUGZILLA_PRODUCT="Red Hat Enterprise Linux 8"
REDHAT_BUGZILLA_PRODUCT_VERSION=8.10
REDHAT_SUPPORT_PRODUCT="Red Hat Enterprise Linux"
REDHAT_SUPPORT_PRODUCT_VERSION="8.10"













podman search ubi9 -> universal base image
podman pull registry.redhat.io/ubi9/ubi9/ubi9
podman login registry.redhat.io 

podman images -> shows iamges of current user
podman ps -> podman running for a current user
podman ps -a -> also shows container no long running
podman run -> pulls the container image and run it. Once the main application stops, the container stops

podman run -d nginx

podman run -it ubi -> inside the container
or
podman run -it ubi /bin/bash -> inside the container


---------Running Rootless Containers-------------
podman run -d -it  registry.access.redhat.com/ubi8/nginx-120
podman inspect -l -f "{{.NetworkSettings.IPAddress}}" -> null (rootless containers doesnt have ip address, they work on ports only)
sudo podman run -d -it  registry.access.redhat.com/ubi8/nginx-120 - has root in it


---------Container Management--------------------
podman ps -a 
podman stop -> stops the container
podman start
podman inspect
podman exec -it catainername sh
podman run -it contaiername sh
ctrl-p,ctrl-q detaches from current interactive session
podman run --name myweb


---------Container Networking--------------------
-> rootless containers cannot be exposed on priviliged ports
-> in port forwarding, a source ip address can be specified to allow access only if traffic comes from a specific ip address : 127.0.0.1:8080:80 nginx

-----Restricting containers----------------------
- in linux, cgroups are used to implement resource restriction
- To run a container with resource restrictions, specific arguments can be used while running the container	
	- sudo podman run --memory=1G --cpu-shares=512 -d bitnami/nginx:latest	
	
	
	
-----------Container Images----------------------- 
podman inspect image ubi | less
pomdan image tree mariadb -> prints the layers
podman image list -> list avaiable images
podman image inspect #shows image details
podman image rm #removes specific images
podman image prune #removes unused images
podman image tag container:1.0 -> copy the image with different tag


----------Create new Image from running container------
podman pull image
podman run -it image sh
	- touch /tmp/testfile
podman commit container-id localhost/image:local



----------Creattin custom container file------------
FROM is used to refer the base image
UBI-Uversal Base image

FROM - Base Image
WORKDIR - sets working directory for next instructions
COPY - copies files from the host to the image
ADD - copies files from host or URL, or unpacks files files from tar archives
RUN - run a command while building the image. Files created by using RUN exit n the file image created.
ENTRYPOINT - defines default command to use. Commnonly set to a shell
CMD - specifies a command to run while starting the resulting image

USER - defines the user account that runs the container commands.

ContainerFile
FROM registry.redhat.io/ubi9/ubi 
RUN dnf install -y nmap
CMD echo hello world

podman build -t myapp .

--------------Best Practices-------------------------
- Use a base image that contais what you need and not much more
- Consider defining a WORKDIR that will be used as the working environment while building the container image
- avoid using unnecessary run statements as each will add another layer to the container image




-----------Entrypoint vs Command---------------------



-------------ARG vs ENV------------------------
ARG

FROM ubi8
ARG arguser
ENV user=${arguser}
RUN mkdir /test && \
	touch /test/${user}
	
CMD ls -l /test


-------Managing Containers UserId-------------

- Podman containers can be started as rootless containers
 - the containerzied process doesn't use the host linux root user
 - the root user inside the container is mapped with only the limited user outside of the container 
 - the container runtime doesn't need to be started by the root user 
- When podman is started by non-priviliged user, it is default to rootless
- After using RUN useradd with the containerfile, 
- the container userid 0 maps to the UID of the user, who started the containers


------Create Imgas lab------
Containerfile  countdown
[student@workstation lab]$ cat Containerfile 
FROM docker.io/redhat/ubi9
COPY countdown .
ENTRYPOINT ["./countdown"]
CMD ["1"]

podman build -t lab4 .
podman run lab4 2

 podman pull docker.io/library/registry:latest
 
 podman tag localhost/lab4 localhost:5000/lab4
 podman push localhost:5000/lab4
 
 podman image tag LOCAL_IMAGE:TAG LOCAL_IMAGE:NEW_TAG
 
 
 
 
 ------------Persistant storage---------------
 podman image tree <imagename> -> to investigate the image layers

Either -v (--volume) or --mount option can be used to mount persistant data
 -mkdir /web
 -podman run -d --port 8080:8080 \
	--volume /web:/var/www/html:ro <image-id>
	
	
---volumes-----------
- volumes are independent objects managed by podman
- podman volume create <volume-name>
- they can refer to localhost based storage
- as volumes independently manages, it can shared between the different containers
- no additional arguments are needed when creating volume


- podman volume import  #to import the archived data
- podman volume export --output # to export the data from volume


-------Managing Permissions-------
- when a continer is started, it starts with a new namespace with its own UID
- To have this working correctly, the container UID must have the permissions in that namespace
- To monitor these permissions, use podman unshare 
 - podman unshare ls -ls /web

- podman run -d -v /home/student/dbfiles:/var/lib/mysql:Z -e MYSQL_ROOT_PASSWORD=password registry.access.redhat.com/rhscl/mysql-80-rhel7
  (this command bind the volume from student directory to the containers directory) 
  
 -------Managing SELinux----------
 
 
 
 -----------Trouble Shooting------------
 
 - podman ps -a [to verify what happenend to the container while starting]
 - podman logs
 
 ---Networking Issues----
 
 - Containers are accessed by the mapping the port on the host to the port used by the container application
 - if accessing the host porst doesn't work, find out the port on which the application is running
 - To do so, use podman exec -it CONTAINER ss -pant
 - Notice that not all containers expose a port externally, some containers connect to and internal network only
  - in that case, use podman inspect CONTAINER --format='{{NetworkSettings.Networks}}'
  
  
  -----connecting to running container---------
podman exec to connect the running containers  







--------------Integrating Podman containers with systemd------------

--start=alwasy (use when starting a container use, so it retarts automcaitclay if anything happens)
- to have containers with this restart policy started on system boot, make sure you run the required systemmd services
	- systemctl enable --now podman-restart
	- systemctl --user enable --now podman-restart

podman run -d --name web nginx
podman generate systemd --name web --files
container-web.service

-> user unit files always will be stored under  ~/.config/systemd/user/
-> tor automatically start a user unit file, systemctl enable --user 
-> without user to login, the linger feature must be enables for the user
-> loginctl enable-linger username 

podman generate systemd --name containername -n --new --files

podman run -d -p 8082:8080 --name mynginx docker.io/library/nginx
podman generate systemd mynginx --files -n
sudo cp container-mynginx.service /lib/systemd/user/	
systemctl --user status container-mynginx
systemctl --user enable container-mynginx

podman auto-update
 podman run -d --label "io.containers.autoupdate=registry" -p 8083:8080 --name autonginx docker.io/library/nginx
 podman generate systemd --name autonginx --files -n [because podman only has the access]
  /home/ashwin/.config/systemd/user

	
	
	
	
	
	
	
	
	
-------------toucble shooting------------

podman exec CONTAINER ss -pant

-p: display the process using the socket

-a: display listening and established connections

-n: display numeric ports instead of mapped service names

-t: display TCP sockets

podman inspect CONTAINER --format '{{.State.Pid}}' [to get the container id]

 sudo nsenter -n -t 525 ss -pant 
 
 
-------------docker events--------

podman info --format {{.Host.EventLogger}}
podman events [live logs]
podman events --filter event=create --filter type=container --stream=false
--------Microservices in Podman-----------

-	podman-compose use YAML file compose.yaml that defines one ore  ore containers with their required properteis
- podman compose is not recommended, in production use kubernetes and openshift
- podman generate kube to generate these YAML files from existing containers
- podman play kube to create containers based on kubernetes yaml files

- podman-compose up - to process the compose file instructions
- podman-compose up -d [detached mode]

-Compose.yml
 - services [these are the containers with all of their parameters. Multiples services can be created as child elements of the servces paramater]
 - volumes [allows for automated creation of storage volumes]
 - netword [allows for creation of dedicated networks]
 
-Container properties
 - image 
 - networks
 - port 

services:
	frontend:
		container_name: frontend
		image: docker.io/library/nginx
		ports:
			- "8080"80"
		networks: datanet
		volumes:
			- datavol:/data
	
	backend:
		image:docker.io/library/mariadb
		container_name: "backend_db"
		networks: datanet
		environment:
			MARIADB_ROOT_PASSWORD: password

networks;
  datanet:{}
volumes:
  datavol:{}  
  
  
podman-compose up  
  
  
  
  
----------Composefile Labs-----------

services:
	fro
	
podman-compose down -v [removes volume also]

networks:
  app-net: {}
  db-net: {}

frontend:
  networks:
    - app-net

volumes:
  my-volume:
    external: true
for already created volume 

Command	Purpose
podman-compose up	create + start
podman-compose up -d	background
podman-compose stop	stop only
podman-compose down	remove containers
podman-compose down -v	remove volumes


services:
  db:
    image: "registry.ocp4.example.com:8443/rhel9/postgresql-13:1"
    environment:
      POSTGRESQL_USER: backend
      POSTGRESQL_DATABASE: rpi-store
      POSTGRESQL_PASSWORD: redhat
    ports:
      - "5432:5432"
    volumes:
      - ./database_scripts:/opt/app-root/src/postgresql-start:Z
      - rpi:/var/lib/pgsql/data

volumes:
  rpi: {}
	
	
--------Orchestration---------
- The pods are should be started through a Deployment, to provide scalability	

- oc run nginx --image=bitnami/nginx
- oc get all
- oc delete pods nginx
- oc create deploy webshop --image=bitnami/nginx --replicas=3






name: compose-lab

services:
  wiremock:
    container_name: quotes-provider
    image: registry.ocp4.example.com:8443/redhattraining/wiremock
    volumes:
      - ~/DO188/labs/compose-lab/wiremock/stubs:/home/wiremock:Z
    networks:
      - backend-net     
  # Define quotes-provider
  quotes-api:
    container_name: quotes-api
    image: registry.ocp4.example.com:8443/redhattraining/podman-quotesapi-compose
    networks:
      - backend-net
      - frontend-net
    environment:
      QUOTES_SERVICE: http://quotes-provider:8080    
  # Define quotes-api
  quotes-ui:
    container_name: quotes-ui
    image: registry.ocp4.example.com:8443/redhattraining/podman-quotes-ui
    networks:
      - frontend-net
    ports:
      - 3000:8080    
  # Define quotes-ui
  #
networks:
  backend-net: {}
  frontend-net: {}


  


----------learn about nsenter-------
nsenter is a host-level Linux tool that lets you enter the namespaces of another process.

podman inspect 81f --format="{{.State.Pid}}"
sudo nsenter -n -t 5865 free









----------- Openshift-----------

oc new-project <projectname>
oc get pod
oc create -f pod.yaml
oc delete pod quotes-ui
oc logs react-ui


oc create deploy webshop --image=bitnami/nginx --replicas=3
oc create deployment gitea --port 3030 --image=registry.ocp4.example.com:8443/redhattraining/podman-gitea:latest
oc get pods
oc create deployment gitea-postgres --port 5432 -o yaml --image registry.ocp4.example.com:8443/rhel9/postgresql-13:1 --dry-run=client > postgres.yaml
oc create deployment gitea-postgres --port 5432 -o yaml --image registry.ocp4.example.com:8443/rhel9/postgresql-13:1 --dry-run=client > postgres.yaml


apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: gitea-postgres
  name: gitea-postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: gitea-postgres
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: gitea-postgres
    spec:
      containers:
      - image: registry.ocp4.example.com:8443/rhel9/postgresql-13:1
        name: postgresql-13
        ports:
        - containerPort: 5432
        env:
        - name: POSTGRESQL_USER
          value: gitea
        - name: POSTGRESQL_PASSWORD
          value: gitea
        - name: POSTGRESQL_DATABASE
          value: gitea
		  
		  
oc expose deployment gitea-postgres		
oc expose service gitea  



-------------labs-----------
oc new-project my-project
oc get projects
oc create -f postgres.yaml
oc delete -f service.yaml
oc delete pod hello-client

Internal access	oc expose deployment <name>
External access	oc expose service <name>
