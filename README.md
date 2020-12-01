# JAMStack App Deployment
Installation steps to enable multi JAMStack apps using Ubuntu, Multipass, Nginx, PM2, Nuxt

## Steps:
1. [Create Ubuntu instance using Multipass](#step-1-create-ubuntu-instance-using-multipass)
2. [Configure Ubuntu instance - Firewall & Nginx](#step-2-configure-ubuntu-instance-firewall-nginx)
3. [Install, Create & Configure Docker Container in Ubuntu instance](#step-3-install-create-configure-docker-container-in-ubuntu-instance)
4. [Configure Docker Container - Nodejs, NPM](#step-4-configure-docker-container-nodejs-npm)
5. [Create new Docker image](#step-5-create-new-docker-image)
6. [Create Nuxt App in Docker Container](#step-6-create-nuxt-app-in-docker-container)
7. [Configure NGINX in Ubuntu instance](#step-7-configure-nginx-in-ubuntu-instance)
8. [Configure Nuxt app in Docker container](#step-8-configure-nuxt-app-in-docker-container)
9. [Test Nuxt App from host](#step-9-test-nuxt-app-from-host)
10.  [Install, Configure and Run PM2 in Docker container](#step-10-install-configure-and-run-pm2-in-docker-container)


## Step 1: Create Ubuntu instance using Multipass

<p align="center">
  <img src="https://dashboard.snapcraft.io/site_media/appmedia/2019/05/multipass.png" width="50" title=""> Multipass is an instance Ubuntu VMs.
  It is a mini-cloud on your workstation using native hypervisors of all the supported plaforms (Windows, macOS and Linux)
</p>

### Step 1.1: Download and install multipass - https://multipass.run/

After downloaded and installed Multipass, run the folowing command to create ubuntu instance on your workstation

***For Windows users, use Windows Powershell (admin) to run below commands #Right Click Start***

*  to check multipass version
```
% multipass version
```
*  to create and launch Ubuntu instance named **ubuntu-01** with **5GB storage** and **2GB memory**
```
% multipass launch --disk 5G --mem 2G --name ubuntu-01

```
*  to list running ubuntu
```
% multipass list
```
*  to access ubuntu instance
```
% multipass shell ubuntu-01
```

*  once inside ubuntu-01 instance, you should use **sudo** to run command.
*  Update and install necessary Ubuntu components
```
$ sudo apt-get update && sudo apt-get upgrade -y
$ sudo apt-get install build-essential -y
```

## Step 2: Configure Ubuntu instance - Nginx Server

### Step 2.1: Install NGINX web server
```
$ sudo apt install nginx
```

*  to check NGINX service status
```
$ sudo systemctl status nginx
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2020-10-27 17:49:22 +08; 1min 38s ago
       Docs: man:nginx(8)
   Main PID: 16279 (nginx)
      Tasks: 2 (limit: 3551)
     Memory: 3.7M
     CGroup: /system.slice/nginx.service
             ├─16279 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
             └─16280 nginx: worker process

Oct 27 17:49:22 ubuntu-01 systemd[1]: Starting A high performance web server and a reverse proxy server...
Oct 27 17:49:22 ubuntu-01 systemd[1]: Started A high performance web server and a reverse proxy server.
```

## Step 3: Install, Create & Configure Docker Container in Ubuntu instance

### Step 3.1: Install Docker in ubuntu-01
*  install curl
```
$ sudo apt install curl
```
*  Download and install docker at ubuntu-01
```
$ curl -sSL https://get.docker.com/ | sh
```
*  To use Docker as a non-root user, you should now consider adding your user to the "docker" group with something like:
```
$ sudo usermod -aG docker $USER
```
*  Exit from ubuntu-01 instance for the configuration to take effect
```
$ exit
logout
```
*  From ***host*** access to ubuntu-01 instance again
```
% multipass shell ubuntu-01
```
*  list and check docker version
```
$ docker ps
CONTAINER ID   STATUS   IMAGE   PORTS   NAMES

$ docker version
Client: Docker Engine - Community
 Version:           19.03.13
 ...
Server: Docker Engine - Community
 Engine:
  Version:          19.03.13
 ...
```

*  Check docker service status
```
$ sudo systemctl status docker
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2020-10-27 17:59:36 +08; 10min ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 17671 (dockerd)
      Tasks: 8
     Memory: 35.3M
     CGroup: /system.slice/docker.service
             └─17671 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
 ...
```

### Step 3.2: Create & Configure Docker in ubuntu-01

*  To create ubuntu docker container that does not exit when launch named **ubuntu-app** with **2GB Memory**
```
$ docker run -d --memory="2g" --name ubuntu-app  ubuntu  bash -c "shuf -i 1-10000 -n 1 -o /data.txt && tail -f /dev/null"

$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
07da039803a4        ubuntu              "bash -c 'shuf -i 1-…"   27 seconds ago      Up 26 seconds                           ubuntu-app
```
*  To acces docker container = ubuntu-app
```
$ docker exec -it ubuntu-app /bin/bash
```
*  Update docker container
```
$ apt-get update && apt-get upgrade -y
$ apt-get install build-essential -y
$ apt-get install curl nano
```

## Step 4: Create new Docker image
This step will create a **new Docker image** based on the updated Docker container = ubuntu-app which include **Firewall, NGINX, Node.js and NPM**

### Step 4.1: Commit Docker container
At **ubuntu-01** commit the existing docker container = **ubuntu-app** to new docker image = **ubuntu-node**
```
$ docker commit ubuntu-app ubuntu-node

$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu-node         latest              f67e1466afcc        3 seconds ago       441MB
ubuntu              latest              d70eaf7277ea        3 days ago          72.9MB
```
### Step 4.2: Create new Docker container
At **ubuntu-01** create new container named **nuxtapp** from docker image **ubuntu-node** that exposes port **3000**
```
$ docker run -p 3000:3000 -td --name nuxtapp ubuntu-node

$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
87f809e6c0b1        ubuntu-node         "bash -c 'shuf -i 1-…"   3 seconds ago       Up 3 seconds        0.0.0.0:3000->3000/tcp   nuxtapp
```

## Step 5: Configure NGINX in Ubuntu instance
At **ubuntu-01** instance, execute the following steps
% 

### Step 5.1: Get Docker container IP Address
*  To get nuxtapp docker container IP address
```
$ docker inspect nuxtapp
...
    "Gateway": "172.17.0.1",
    "IPAddress": "172.17.0.3",
    "IPPrefixLen": 16,
...
```
The IP Address is **172.17.0.3**


### Step 5.2: Configure NGINX location directives
Using the location directive to **HTTP proxy** nuxt app, so from host we can call **http://localhost/portal** 
*  Edit file default in /etc/nginx/sites-enabled/
```
$ cd /etc/nginx/sites-enabled/
$ sudo nano default
```
*  In default file, add the following new location directive = /portal
```
...
location /portal {
	proxy_http_version 1.1;
        proxy_pass http://172.17.0.3:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
	}
...

Ctrl-o and press enter to save the file
Ctrl-x to exit
```
*  Verify NGINX configuration
```
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

*  restart NGINX
```
$ sudo systemctl restart nginx
```
*  check NGINX service status
```
$ sudo systemctl status nginx
```
