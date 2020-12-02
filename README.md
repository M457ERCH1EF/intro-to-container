# JAMStack App Deployment
Installation steps to enable multi JAMStack apps using Ubuntu, Multipass, Nginx, PM2, Nuxt

## Steps:
1. [Create Ubuntu instance using Multipass]
2. [Configure Ubuntu instance - Firewall & Nginx]
3. [Install, Create & Configure Docker Container in Ubuntu instance]
4. [Configure Docker Container - Nodejs, NPM]
5. [Create new Docker image]
6. [Clone Web App in Docker Container from GitHub]
7. [Configure NGINX in Ubuntu instance]
8. [Configure Nuxt app in Docker container]
9. [Test Nuxt App from host]


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

## Step 2: Configure Ubuntu instance - Nginx

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

*  To create ubuntu docker container that does not exit when launch named **ubuntu-app** with **1GB Memory**
```
$ docker run -d --memory="1g" --name ubuntu-app  ubuntu  bash -c "shuf -i 1-10000 -n 1 -o /data.txt && tail -f /dev/null"

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
```
* Install Curl, Nano and Git components
```
$ apt-get install curl nano git
```
## Step 4: Configure Docker Container - Nodejs, NPM

### Step 4.1: Install Node.js in Docker container = **ubuntu-app**

*  To install Node.js v12.x:
```
$ curl -sL https://deb.nodesource.com/setup_12.x -o nodesource_setup.sh

$ bash nodesource_setup.sh

$ apt install nodejs

$ node -v
v12.19.0

$ npm -v
6.14.8

$ exit
```

## Step 5: Create new Docker image
This step will create a **new Docker image** based on the updated Docker container = ubuntu-app which include **NGINX, Node.js and NPM**

### Step 5.1: Commit Docker container
At **ubuntu-01** commit the existing docker container = **ubuntu-app** to new docker image = **ubuntu-node**
```
$ docker commit ubuntu-app ubuntu-node

$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu-node         latest              f67e1466afcc        3 seconds ago       441MB
ubuntu              latest              d70eaf7277ea        3 days ago          72.9MB
```
### Step 5.2: Create new Docker container
At **ubuntu-01** create new container named **nuxtapp** from docker image **ubuntu-node** that exposes port **3000**
```
$ docker run -p 3000:3000 -td --name nuxtapp ubuntu-node

$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
87f809e6c0b1        ubuntu-node         "bash -c 'shuf -i 1-…"   3 seconds ago       Up 3 seconds        0.0.0.0:3000->3000/tcp   nuxtapp
```

## Step 6: Install & Configure web app 
### Step 6.1: Clone Web App from GitHub repository

*  Access to the new Docker container = **nuxtapp**
```
$ docker exec -it nuxtapp /bin/bash
or
$ docker exec -it nuxtapp bash
```
*  clone web app with git
```
$ cd /home
$ git clone https://github.com/M457ERCH1EF/container-app.git
```
* Go to web app folder
```
$ cd container-app
```
*  Install web app
```
$ npm install
```
* Run the app
```
$ npm run dev
...
   ╭───────────────────────────────────────╮
   │                                       │
   │   Nuxt.js @ v2.14.7                   │
   │                                       │
   │   ▸ Environment: development          │
   │   ▸ Rendering:   client-side          │
   │   ▸ Target:      server               │
   │                                       │
   │   Listening: http://localhost:3000/   │
   │                                       │
   ╰───────────────────────────────────────╯
...
Ctrl-c to stop the app
$ exit
```

## Step 7: Configure NGINX in Ubuntu instance
At **ubuntu-01** instance, execute the following steps
% 

### Step 7.1: Get Docker container IP Address
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


### Step 7.2: Configure NGINX location directives
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

## Step 8: Configure Nuxt app in Docker container
### Step 8.1: Edit nuxt.config.js
*  Access to the new Docker container = **nuxtapp**
```
$ docker exec -it nuxtapp /bin/bash
```
*  Goto project folder
```
$ cd /home/project/myapp
```
*  Edit **nuxt.config.js** file
```
$ nano nuxt.config.js
```
*  Add the following script
```
...
server: { 
    port: 3000, // default: 3000
    host: '0.0.0.0' // default: localhost
},
router: {
    base: '/portal/'  //to enable www.domain.com/portal
},
...

Ctrl-o and press enter to save the file
Ctrl-x to exit
```
*  Run the app
```
$ npm run dev
...
   ╭───────────────────────────────────────────────╮
   │                                               │
   │   Nuxt.js @ v2.14.7                           │
   │                                               │
   │   ▸ Environment: development                  │
   │   ▸ Rendering:   client-side                  │
   │   ▸ Target:      server                       │
   │                                               │
   │   Listening: http://172.17.0.3:3000/portal/   │
   │                                               │
   ╰───────────────────────────────────────────────╯
...
```

## Step 9: Test Nuxt App from host
### Step 9.1: Get Ubuntu instance IP Address
*  Open new terminal at host, and run the following command
```
$ multipass list
Name                    State             IPv4             Image
ubuntu-01               Running           192.168.64.3     Ubuntu 20.04 LTS
```
The **ubuntu-01** IP Address is **192.168.64.3**

### Step 9.2: Access nuxt app from browser
*  Open browser - http://192.168.64.3

*  Open browser - http://192.168.64.3/portal
