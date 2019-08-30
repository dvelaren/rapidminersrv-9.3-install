# RapidMiner Server 9.3 Install

This is a guide on how to install RapidMiner 9.3 Server on Ubuntu 18.04

## Requirements
- OS: Ubuntu 18.04 LTS
- CPU: Minimum 2 cores at 2.4 GHz
- RAM: 6 GB
- HDD: 10 GB

## Installation

## Java JRE 1.8

Download Java Run-time Engine from [Java webpage](https://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html)

Change directory to `usr/lib/jvm`
```
cd /usr/lib/jvm
```

Extract the .zip
```sh
sudo tar -xvzf ~/Downloads/jdk-8u221-linux-x64.tar.gz
```

Edit enviroment variables
```sh
sudo nano /etc/environment
```

Update existing PATH variable with the new JRE bin directory. Add a new colon `:` at the end of the PATH and then put the jre directory
```sh
/usr/lib/jvm/jre1.8.0_221/bin
```

Add `JAVA_HOME` environment variable at the end of the environment file
```sh
JAVA_HOME="/usr/lib/jvm/jre1.8.0_221/bin"
```

Save changes of environment file and reload it using source
```sh
source /etc/environment
```

Inform Ubuntu about the installed location using update-alternatives

```sh
sudo update-alternatives --install "/usr/bin/java" "java" "/usr/lib/jvm/jre1.8.0_221/bin/java" 0
```

Configure which version of java will be used in the Ubuntu OS, make sure to select the installed java runtime engine
```sh
sudo update-alternatives --config java
```

Check if java CLI works, the following command should output installed version
```sh
java -version
```

## Installing PostgreSQL

Install PostgreSQL using apt
```sh
sudo apt-get update
sudo apt install postgresql postgresql-contrib
```

Login as postgres user and change default password
```sh
sudo su postgres
```

Access postgres CLI
```sh
psql
```

Change password and set it to something you remember
```sh
\password
```

Create rapidminer schema and db user with required permisions
```sh
CREATE DATABASE rapidminer_server;
CREATE USER rapidminer WITH ENCRYPTED PASSWORD 'rapidminer';
GRANT ALL PRIVILEGES ON DATABASE rapidminer_server TO rapidminer;
```

## Install RapidMiner Server

Create a user account in Ubuntu for rapidminer. Enter as password rapidminer, and as Full Name rapidminer. Leave the other fields empty
```sh
sudo adduser rapidminer
```

Change current Ubuntu user to rapidminer (using GUI)

Download RapidMiner Server from Downloads section inside your RapidMiner account
![alt text](https://docs.rapidminer.com/latest/server/installation/img/download-link-1.png)

Extract rapidminer-server-installer-9.3.2.zip
```sh
unzip rapidminer-server-installer-9.3.2.zip
```

Navigate to the new folder of rapidminer bin directory
```sh
cd /rapidminer-server-installer-9.3.2/bin
```

Execute installer
```sh
. rapidminer-server-installer
```

Continue through the setup until Locations form. Fill it as follows for Install directory and Home directory
- Install directory:
`/home/rapidminer/rapidminer-server/rapidminer-server-installer-9.3.2`
- Home directory:
`/home/rapidminer/rapidminer-server/rapidminer-server-home`

From your account portal Licenses page, copy the license key to your clipboard. (See this additional information if you need help copying your key.)

Configure Server Settings as follows:
- Hostname: yourhostname.example.com
- Port for web interface: 8888
- Server Memory (in MB): 2048
- Number of bundled Job Containers: 1
- Internal port: 5672
- Memory per Job Container (in MB): 2048

Configure Database as follows
- Hostname: localhost
- Port: 5432
- Database schema: rapidminer_server
- Database username: rapidminer
- Database password: rapidminer

Click try connection to check that it's correctly working

Then click next and follow until the end of the installation.

## Executing RapidMiner Server

Navigate to rapidminer install directory and run `standalone.sh` script

```sh
cd /home/rapidminer/rapidminer-server/rapidminer-server-installer-9.3.2/bin
. standalone.sh
```

Wait until all the commands finish and then open a navigator and go to `yourdomain:8888`

Log in with default rapidminer credentials:
- Username: admin
- Password: changeit
  
Click in your account at the top right of the webpage and change default password

Add a new user through the left menu inside Administration/User Management
- Username: rapidminer
- Password: rapidminer

Add rapidminer user to administrator group by clicking on the Display name

## Creating a system service to automatically deploy rapidminer-server on system start

Create a new service inside /etc/init.d/
```sh
sudo nano /etc/init.d/rapidminerserver
```

Fill the contents of that file with the following lines:
```sh
\#!/bin/bash
\### BEGIN INIT INFO
\# Provides:          rapidminerserver
\# Required-Start:    $local_fs $remote_fs $network $syslog
\# Required-Stop:     $local_fs $remote_fs $network $syslog
\# Default-Start:     3 4 5
\# Default-Stop:      0 1 2 6
\# Short-Description: Start/Stop RapidMiner Server
\### END INIT INFO
\# chkconfig: 345 85 15

SERVER_HOME=/home/rapidminer/rapidminer-server/rapidminer-server-installer-9.3.2
SERVER_USER=rapidminer

case "$1" in
    start)
        echo "Starting RapidMiner Server..."
        start-stop-daemon --start --quiet --background --chuid ${SERVER_USER} --exec ${SERVER_HOME}/bin/standalone.sh
    ;;
    stop)
        echo "Stopping RapidMiner Server..."
        start-stop-daemon --start --quiet --background --chuid ${SERVER_USER} --exec ${SERVER_HOME}/bin/jboss-cli.sh -- --connect --command=:shutdown
    ;;
    *)
        echo "Usage: /etc/init.d/rapidminerserver {start|stop}"; exit 1;
    ;;
esac

exit 0
```
Make the current service executable
```sh
sudo chmod 755 /etc/init.d/rapidminerserver
```

Tell Ubuntu to run the service on system start
```sh
update-rc.d rapidminerserver defaults
```

Reboot Ubuntu OS
```sh
sudo shutdown -r now
```

After reboot, open a terminal and check if the rapidminerserver service is running and active
```sh
sudo service rapidminerserver status
```

Navigate again to `http://localhost:8888` and verify server is working
