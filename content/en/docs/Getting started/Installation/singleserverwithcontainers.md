---
title: "Single server with containers"
linkTitle: "Single server with containers"
weight: 10
description: >
  Note: This content is currently in draft status.
tags: ["subtopic"]
---
A single server installation is recommended if you have a single tenant or you can estimate beforehand the number of tenants and resources that your FOLIO instance will handle.  

![FOLIO Single Server components](/img/single_docker_compose.png)

A FOLIO instance is divided into two main components.  The first component is Okapi, the gateway.  The second component is the UI layer which is called Stripes.  The single server with containers installation method will install both.

## System requirements

**Software requirements**

| **Requirement**      | **Recommended Version**                    |
|----------------------|--------------------------------------------|
| Operating system     | Ubuntu 20.04 LTS (Focal Fossa) 64-bits     |
| Java                 | OpenJDK 11                                 |
| PostgreSQL           | PostgreSQL 10                              |

**Hardware requirements**

| **Requirement** | **FOLIO Base Apps** | **FOLIO Extended Apps** |
|-----------------|---------------------|-------------------------|
| RAM             | 12GB                | 20GB                    |
| CPU             | 4                   | 8                       |

## Installing Okapi

### Okapi requirements

1. Update the APT cache.

```
sudo apt-get update
```

2. Install Java 11 and nginx and verify that Java 11 is the system default.
```
sudo apt-get -y install openjdk-11-jdk nginx
sudo update-java-alternatives --jre-headless --jre --set java-1.11.0-openjdk-amd64
```

3. Import the PostgreSQL signing key, add the PostgreSQL apt repository and install PostgreSQL.
```
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo add-apt-repository "deb http://apt.postgresql.org/pub/repos/apt/ focal-pgdg main"
sudo apt-get update
sudo apt-get -y install postgresql-10 postgresql-client-10 postgresql-contrib-10 libpq-dev
```

4. Configure PostgreSQL to listen on all interfaces and allow connections from all addresses (to allow Docker connections).

* Edit the file **/etc/postgresql/10/main/postgresql.conf** to add line **listen_addresses = '*'** in the "Connection Settings" section.
* Edit the file **/etc/postgresql/10/main/postgresql.conf** to increase **max_connections** (e.g. to 500)
* Edit the file **/etc/postgresql/10/main/pg_hba.conf** to add line **host all all 0.0.0.0/0 md5**
* Restart PostgreSQL with command **sudo systemctl restart postgresql**

5. Import the Docker signing key, add the Docker apt repository and install the Docker engine.
```
sudo apt-get -y install apt-transport-https ca-certificates gnupg-agent software-properties-common
wget --quiet -O - https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get -y install docker-ce docker-ce-cli containerd.io
```

6. Configure Docker engine to listen on network socket.

- Create a configuration folder for Docker if it does not exist.  

```
sudo mkdir -p /etc/systemd/system/docker.service.d
```

- Create a configuration file **/etc/systemd/system/docker.service.d/docker-opts.conf** with the following content.

```
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://127.0.0.1:4243
```

- Restart Docker.

```
sudo systemctl daemon-reload
sudo systemctl restart docker
```

7. Install docker-compose.

Follow the instructions from official documentation for [docker](https://docs.docker.com/compose/install/). The instructions may vary depending on the architecture and operating system of your server, but in most cases the following commands will work.

```
sudo curl -L \
  "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" \
  -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

8. Install Apache Kafka and Apache ZooKeeper.  Apache Kafka and Apache ZooKeeper are required by FOLIO [mod-pubsub](https://github.com/folio-org/mod-pubsub).  Both Kafka and ZoopKeepr are installed below using docker-compose.

```
sudo mkdir /opt/kafka-zk
sudo cp /home/user/docker-compose-kafka-zk.yml /opt/kafka-zk/docker-compose.yml
cd /opt/kafka-zk
sudo docker-compose up -d
```

A sample docker-compose (docker-compose-kafka-zk.yml) file can be downloaded from [here](https://github.com/folio-org/folio-install/blob/master/runbooks/single-server/scripts/docker-compose-kafka-zk.yml). Take into account that you have to change the **KAFKA_INTER_BROKER_LISTENER_NAME** value for the private IP of your server. 


### Create a database and role for Okapi

You will need to create one database in PostgreSQL to persist the Okapi configuration.

1. Log into the PostgreSQL server as a superuser.

```
sudo su -c psql postgres postgres
```

2. Create a database role for Okapi and a database to persist Okapi configuration.
```
CREATE ROLE okapi WITH PASSWORD 'okapi25' LOGIN CREATEDB;
CREATE DATABASE okapi WITH OWNER okapi;
```

3. Exit psql with **\q** command

**Note**: You will need to create additional databases for each new tenant you add to FOLIO. More information on how to set up a new tenants on the next sections.

### Install and configure Okapi

Once you have installed the requirements for Okapi and created a database, you can proceed with the installation.  Okapi is available as a DEB package that can be easily installed in Debian-based operating systems. You only need to add the official APT repository to your server.

1. Import the FOLIO signing key, add the FOLIO apt repository and install okapi.

```
wget --quiet -O - https://repository.folio.org/packages/debian/folio-apt-archive-key.asc | sudo apt-key add -
sudo add-apt-repository "deb https://repository.folio.org/packages/ubuntu focal/"
sudo apt-get update
sudo apt-get -y install okapi=4.3.2-1
sudo apt-mark hold okapi
```

Please note that the last stable version of FOLIO is 4.3.2-1.  If you do not explicitly set the Okapi version, you will install the latest Okapi release.  There is some risk with installing the latest Okapi release.  The latest release may not have been tested with the rest of the components in the quarterly release.

2. Configure Okapi to run as a single node server with persistent storage.

- Edit file **/etc/folio/okapi/okapi.conf** to reflect the following changes: 

```
role="dev"
port_end="9230"
host="10.0.2.15"
storage="postgres"
okapiurl="http://10.0.2.15:9130"
```
**Note 1**: The IP address that you use in the properties **host** and **okapiurl** should match the private IP of your server.  This IP address should be reachable from Docker containers.  Therefore, you can not use localhost.  You can use the /**ifconfig** command in order to determine the private IP. 

**Note 2**: The properties **postgres_host**, **postgres_port**, **postgres_username**, **postgres_password**, **postgres_database** should be configured in order to match the PostgreSQL configurations made previously.

3. Restart Okapi

```
sudo systemctl daemon-reload
sudo systemctl restart okapi
```
The Okapi log is at **/var/log/folio/okapi/okapi.log**.

4. Pull module descriptors from the central registry.

A module descriptor declares the basic module metadata (id, name, etc.), specifies the module's dependencies on other modules (interface identifiers to be precise), and reports all "provided" interfaces. As part of the continuous integration process, each Module Descriptor  is published to the FOLIO Registry at https://folio-registry.aws.indexdata.com/.

```
curl -w '\n' -D - -X POST -H "Content-type: application/json" \
  -d @registry.json \
  http://localhost:9130/_/proxy/pull/modules
```
The content of registry.json should look like this:

```
{
  "urls": [
    "http://folio-registry.aws.indexdata.com"
  ]
}
```

Okapi is up and running!

Now that you have an Okapi instance, you can proceed to install Stripes.  However, Stripes is bundled and deployed on a per tenant basis.  So, you have to decide whether to install platform-core or platform-complete for your tenant.

### Create a new tenant

1. Create a database for your tenant.  This database will host the data of your tenant.

- Log into the PostgreSQL server as a superuser.

```
sudo su -c psql postgres postgres
```
- Create a database role for Okapi and a database to persist Okapi configuration.

```
CREATE ROLE folio WITH PASSWORD 'folio123' LOGIN CREATEDB;
CREATE DATABASE folio WITH OWNER folio;
```
- Exit psql with **\q** command

2. Create a new tenant on Okapi.

- Post the tenant initialization to Okapi.
```
curl -w '\n' -D - -X POST -H "Content-type: application/json" \
  -d @tenant.json \
  http://localhost:9130/_/proxy/tenants
```
The content of tenant.json:
```
{
  "id" : "diku",
  "name" : "Datalogisk Institut",
  "description" : "Danish Library Technology Institute"
}
```

**Note**:  In this installation guide, the Datalogisk Institut is used as an example, but you should use the information for your organization.  Take into account that you have to use the id of your tenant in the next steps.

- Next, enable the Okapi internal module for the tenant

```
curl -w '\n' -D - -X POST -H "Content-type: application/json" \
  -d '{"id":"okapi"}' \
  http://localhost:9130/_/proxy/tenants/diku/modules
```
3. Post data source information to the Okapi environment for use by deployed modules.
```
curl -w '\n' -D - -X POST -H "Content-Type: application/json" -d "{\"name\":\"DB_HOST\",\"value\":\"10.0.2.15\"}" http://localhost:9130/_/env
curl -w '\n' -D - -X POST -H "Content-Type: application/json" -d "{\"name\":\"DB_PORT\",\"value\":\"5432\"}" http://localhost:9130/_/env
curl -w '\n' -D - -X POST -H "Content-Type: application/json" -d "{\"name\":\"DB_DATABASE\",\"value\":\"folio\"}" http://localhost:9130/_/env
curl -w '\n' -D - -X POST -H "Content-Type: application/json" -d "{\"name\":\"DB_USERNAME\",\"value\":\"folio\"}" http://localhost:9130/_/env
curl -w '\n' -D - -X POST -H "Content-Type: application/json" -d "{\"name\":\"DB_PASSWORD\",\"value\":\"folio123\"}" http://localhost:9130/_/env
curl -w '\n' -D - -X POST -H "Content-Type: application/json" -d "{\"name\":\"KAFKA_HOST\",\"value\":\"10.0.2.15\"}" http://localhost:9130/_/env
curl -w '\n' -D - -X POST -H "Content-Type: application/json" -d "{\"name\":\"OKAPI_URL\",\"value\":\"http://10.0.2.15:9130\"}" http://localhost:9130/_/env
```

**Note**: Make sure that you use your private IP for the properties **DB_HOST**, **KAFKA_HOST** and **OKAPI_URL**. 

4. Decide if you would like to use platform-core or platform-complete for your tenant and clone the repository.  The tenant is now ready to add some Apps.

The App installation process is similar for platform-core and platform-complete.  You have to clone one of these github repositories: https://github.com/folio-org/platform-core or https://github.com/folio-org/platform-complete.

In this installation guide, the ‘platform-core’ repository will be used.  If you would like to install ‘platform-complete’ you should replace every mention of platform-core with platform-complete in the instructions.

- Clone the repository

```
git clone https://github.com/folio-org/platform-core
cd platform-core
```
- Checkout a stable branch of the repository

```
git checkout q3-2020
```

5. Post the list of backend modules to deploy and enable. Also, you can set the (tenantParameters)[https://github.com/folio-org/okapi/blob/master/doc/guide.md#install-modules-per-tenant] to load their sample and reference data.

```
curl -w '\n' -D - -X POST -H "Content-type: application/json" \
  -d @okapi-install.json \
  http://localhost:9130/_/proxy/tenants/diku/install?deploy=true\&preRelease=false\&tenantParameters=loadSample%3Dtrue%2CloadReference%3Dtrue
```

This will take a long time to return because all of the Docker images must be pulled from Docker Hub.  Progress can be followed in the Okapi log at /var/log/folio/okapi/okapi.log

**Note**: You will have to replace ‘diku’ with the id of your tenant.

6. Post the list of Stripes modules to enable.

```
curl -w '\n' -D - -X POST -H "Content-type: application/json" \
  -d @stripes-install.json \
  http://localhost:9130/_/proxy/tenants/diku/install?preRelease=false
```

The backend of the new tenant is ready.  Now, you have to set up a Stripes instance for the frontend of the tenant, create a superuser for the tenant and secure Okapi.


### Create a superuser

You need to create a superuser for the newly created tenant.  This is a multi step process and the details can be found in the (Okapi documentation) [https://github.com/folio-org/okapi/blob/master/doc/guide.md#securing-okapi]. You can use a PERL script to execute these steps automatically.   You only need to provide the tenant id, a username/password for the superuser and the URL of Okapi.

```
perl bootstrap-superuser.pl \
  --tenant diku --user diku_admin --password admin \
  --okapi http://localhost:9130
```

You can download the bootstrap-superuser.pl script (here)[https://github.com/folio-org/folio-install/blob/master/runbooks/single-server/scripts/bootstrap-superuser.pl ].

### Secure Okapi

By default, Okapi API is open in order to facilitate the deployment process of FOLIO. However, in a production environment you must enable the security checks. You can use a Python script to secure Okapi, you should provide a username and password for Okapi.

```
python3 secure-supertenant.py -u USERNAME -p PASSWORD -o http://localhost:9130
```

The script can be downloaded (here)[https://github.com/folio-org/folio-install/blob/master/runbooks/single-server/scripts/secure-supertenant.py].

When Okapi is secured, you must login using **mod-authtoken** to obtain an authtoken and include it in the **x-okapi-token** header for every request to the Okapi API.  For example, if you want to repeat any of the calls to Okapi in this guide, you will need to include **x-okapi-token:YOURTOKEN** and **x-okapi-tenant:supertenant** as headers for any requests to the Okapi API.

## Install Stripes


### Stripes requirements

1. Install build requirements from Ubuntu apt repositories.

```
sudo apt-get -y install git curl nodejs npm libjson-perl libwww-perl libuuid-tiny-perl
```

2. Install n from npm.

```
sudo npm install n -g
```

3. Import the Yarn signing key, add the Yarn apt repository and install Yarn.

```
wget --quiet -O - https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
sudo add-apt-repository "deb https://dl.yarnpkg.com/debian/ stable main"
sudo apt-get update
sudo apt-get -y install yarn
```

### Building Stripes

1. Move to NodeJS LTS.

```
sudo n lts
```
2. Clone the platform-core repository and cd into it.

```
git clone https://github.com/folio-org/platform-core
cd platform-core
```
3. Checkout a stable branch of the repository

```
git checkout q3-2020
```

4. Install npm packages.

```
yarn install
```
5. Configure Stripes.

- Edit the file **stripes.config.js** and change **okapi.url** and **okapi.tenant**.

```
...
okapi: { 'url':'http://folio.server.com:9130', 'tenant':'diku' },
..
```
Make sure that you use the public IP or domain of your server since this URL will be used to request Okapi from the clients’ browsers.

6. Build webpack.

```
NODE_ENV=production yarn build output
cd ..
```
A new folder called ‘output’ will be created which contains the Stripes configured webpack of your tenant. 

7.  Server the contents of this output folder on a web server.

### Serve Stripes

Now that the webpack is built, you can configure the 'nginx' server.

1. Define a directory for the Stripes webpacks of the tenants.  For example, you can use **/home/folio/tenants**.
2. Copy the Stripes webpack to the new directory.

```
mkdir /home/folio/tenants/diku
cp -R output/. /home/folio/tenants/diku/
```

3. Configure NGINX to serve this directory.

```
sudo cp nginx-stripes.conf /etc/nginx/sites-available/stripes
sudo ln -s /etc/nginx/sites-available/stripes /etc/nginx/sites-enabled/stripes
sudo rm /etc/nginx/sites-enabled/default
sudo systemctl restart nginx
```
The content of nginx-stripes.conf should look like this:

```
server {
  listen 80;
  server_name folio-server.com;
  charset utf-8;
  # Serve index.html for any request not found
  location / {
    # Set path
    root /home/folio/tenants/diku;
    include mime.types;
    types {
      text/plain lock;
    }
    try_files $uri /index.html;
  }
}
```


You should use your public IP or domain name in the field ‘server_name’. 

**Note**: If you want to host multiple tenants on a server, you can configure NGINX to either open a new port for each tenant or set up different paths on the same port (e.g. /tenat1, /tenant2).

3. Now Stripes is running on the port 80 and you can open it using a browser.


## Install and serve edge modules (platform-complete only)

The Edge modules bridge the gap between some specific third-party services and FOLIO (e.g. RTAC, OAI-PMH).  In these FOLIO reference environments, the set of edge services are accessed via port 8000.  In this example, the edge-oai-pmh will be installed.

You can find more information about the Edge modules of FOLIO in the Wiki https://wiki.folio.org/display/FOLIOtips/Edge+APIs.

1. Create institutional user. An institutional user must be created with appropriate permissions to use the edge module. You can use the included create-user.py to create a user and assign permissions.

```
python3 create-user.py -u instuser -p instpass \
    --permissions oai-pmh.all --tenant diku \
    --admin-user diku_admin --admin-password admin
```
The script can be found (here) [https://github.com/folio-org/folio-install/blob/master/runbooks/single-server/scripts/create-user.py].

If you need to specify an Okapi instance running somewhere other than http://localhost:9130, then add the --okapi-url flag to pass a different url.  If more than one permission set needs to be assigned, then use a comma delimited list, i.e. --permissions edge-rtac.all,edge-oai-pmh.all.

2. The institutional user is created for each tenant for the purposes of edge APIs. The credentials are stored in one of the secure stores and retrieved as needed by the edge API. You can find more information about secure stores (here) [https://github.com/folio-org/edge-common#secure-stores].  In this example, a basic EphemeralStore using an **ephemeral.properties** file which stores credentials in plain text.  This is meant for development and demonstration purposes only.

```
sudo mkdir -p /etc/folio/edge
sudo vi /etc/folio/edge/edge-oai-pmh-ephemeral.properties
```
The ephemeral properties file should look like this.


```
secureStore.type=Ephemeral
# a comma separated list of tenants
tenants=diku
#######################################################
# For each tenant, the institutional user password...
#
# Note: this is intended for development purposes only
#######################################################
# format: tenant=username,password
diku=instuser,instpass
```

3. Start edge module Docker containers.
You will need the version of the edge-modules available on Okapi for the tenant.  You can run a CURL request to Okapi and get the version of the **edge-oai-pmh** module.


```
curl -s http://localhost:9130/_/proxy/tenants/diku/modules | jq -r '.[].id' | grep 'edge-'
```

- Set up a docker compose file in **/etc/folio/edge/docker-compose.yml** that defines each edge module that is to be run as a service. The compose file should look like this.

```
version: '2'
services:
  edge-oai-pmh:
    ports:
      - "9700:8081"
    image: folioorg/edge-oai-pmh:2.2.1
    volumes:
      - /etc/folio/edge:/mnt
    command:
      -"Dokapi_url=http://10.0.2.15:9130"
      -"Dsecure_store_props=/mnt/edge-oai-pmh-ephemeral.properties"
    restart: "always"
```
Make sure you use the private IP of the server for the Okapi URL.


- Start the edge module containers.

```
cd /etc/folio/edge
sudo docker-compose up -d
```

4. Set up NGINX.

- Create a new virtual host configuration to proxy the edge modules.   Create a new NGINX file in the directory **/etc/nginx/sites-available/edge**.

```
server {
  listen 8130;
  server_name localhost;
  charset utf-8;

  location /oai {
    proxy_pass http://localhost:9700;
  }
}

```
- Link that new configuration and restart nginx.

```
sudo ln -s /etc/nginx/sites-available/edge /etc/nginx/sites-enabled/edge
sudo service nginx restart

```

Now, an OAI service is running on http://server:8130/oai . 

5. Follow this procedure to generate the API key for the tenant and institutional user that were configured in the previous sections.  Currently, the edge modules are protected through API Keys.

```
cd ~
git clone https://github.com/folio-org/edge-common.git
cd edge-common
mvn package
java -jar target/edge-common-api-key-utils.jar -g -t diku -u instuser
```

This will return an API key that must be included in requests to edge modules. With this APIKey, you can test the edge module access.  For instance, a test OAI request would look like this.

```
curl -s "http://localhost:8130/oai?apikey=APIKEY=&verb=Identify"
```
The specific method to construct a request for an edge module is documented in the developers website: https://dev.folio.org/source-code/map/ or you can refer to the github project of the edge module.
