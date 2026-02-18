# instal artifactory oss


## linsk and sources

- [downloadlink](https://jfrog.com/community/download-artifactory-oss/)
- [install instructions rhel](https://jfrog.com/help/r/jfrog-installation-setup-documentation/install-artifactory-on-rpm)
- 

## Preflight check

```bash
# download utility [link1](https://jfrog.com/help/r/jfrog-installation-setup-documentation/preflight-check)
# and [link2](https://releases.jfrog.io/ui/repos/tree/General/run/jfrog-diagnostics-util/0.1.1)

wget https://releases.jfrog.io/artifactory/run/jfrog-diagnostics-util/0.1.1/diagnostics-linux-amd64

# after download rename the binary

mv diagnostics-<os>-<arch> diagnosticUtil

# use the util, start with the desired paramater (system, connectivity, database, all)

./diagnosticUtil inspect <subcommand> <options>

# for further information please read the documentation on the links at the begining this section
```



## Install steps

### Preflight check

- check wether OS supported [requirements matrix](https://jfrog.com/help/r/jfrog-installation-setup-documentation/requirements-matrix)

- check [Artifactory System Requirements and Platform Support](https://jfrog.com/help/r/jfrog-installation-setup-documentation/artifactory-system-requirements-and-platform-support)

- VM compatibility check to minimum system requirements [preflight check](https://jfrog.com/help/r/jfrog-installation-setup-documentation/preflight-check

- network requirements [network requirements](https://jfrog.com/help/r/jfrog-installation-setup-documentation/network-requirements-for-jfrog-products)

- additional [general system requirements](https://jfrog.com/help/r/jfrog-installation-setup-documentation/general-system-requirements)

- additional [artifactory system requirements](https://jfrog.com/help/r/jfrog-installation-setup-documentation/artifactory-system-requirements-and-platform-support)

### Install artifactory RPM Package

- first install Java and Postgresql


```bash
# set JFROG_HOME variable

export JFROG_HOME=/opt/jfrog

# for more check [jfrog produkt directory structure](https://jfrog.com/help/r/jfrog-installation-setup-documentation/jfrog-home)

# Configure Jfrog repository file:

wget https://releases.jfrog.io/artifactory/artifactory-rpms/artifactory-rpms.repo -O jfrog-artifactory-rpms.repo;

# move repository file

sudo mv jfrog-artifactory-rpms.repo /etc/yum.repos.d/

# update yum/dnf cache and install artifactroy

sudo yum update && sudo yum install jfrog-artifactory-oss

# on newer rhel and centos versions use the following 

sudo dnf install jfrog-artifactory-oss

# find artifactorie"s system.yaml to edit, and set the RAM usage - (JVM Memory Parameters)

sudo find / -name system.yaml 2>/dev/null

# snippet to set the memory parameters

shared:
  extraJavaOpts: "-Xms512m -Xmx2g"

```

### Download Desired Artifactory Package

- to download a desired version of Artifactory, example copied from jfrog

```bash
# Example for Artifactory Pro (replace with your desired version)
curl -g -L -O -J 'https://releases.jfrog.io/artifactory/artifactory-pro-rpms/jfrog-artifactory-pro/jfrog-artifactory-pro-[RELEASE].rpm'

# For example, to download Artifactory Pro 7.111.11:
curl -g -L -O -J 'https://releases.jfrog.io/artifactory/artifactory-pro-rpms/jfrog-artifactory-pro/jfrog-artifactory-pro-7.111.11.rpm'

## Install .rpm package
# navigate to package where its downloaded and install using yum and root privileges

# Example for Artifactory Pro
sudo yum install -y ./jfrog-artifactory-<pro|oss|cpp-ce>-<VERSION>.rpm

# For example, to install Artifactory Pro 7.111.11
sudo yum install -y ./jfrog-artifactory-pro-7.111.11.rpm

```

### Install Java JDK 21

- important, reserve memory in java for artifactory, according to load, in system.yaml
- see the links 
	- [java requirements](https://jfrog.com/help/r/jfrog-installation-setup-documentation/java-requirements-for-jfrog-products)
	- [system.yaml](https://jfrog.com/help/r/jfrog-installation-setup-documentation/system-yaml-configuration-file)

```bash
# download java 21jdk 

wget https://download.oracle.com/java/21/latest/jdk-21_linux-x64_bin.rpm

# Install the rpm

sudo dnf install ./jdk-21_linux-x64_bin.rpm

# verify

java -version


```


### Set up artifactroy Database

#### Install and Init Postgresql

```bash
sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo dnf install -y postgresql17-server
sudo /usr/pgsql-17/bin/postgresql-17-setup initdb
sudo systemctl enable postgresql-17
sudo systemctl start postgresql-17

# verify

systemctl status postgresql-17.service
```



#### Configure Postgresqls

```bash
# variant a, configure postgresql

# switch to postgres user

sudo -i -u postgres

# enter psql shell

psql

# clean up, recommended

DROP DATABASE IF EXISTS artifactory;
DROP ROLE IF EXISTS artifactory;

# create dedicated artifactory role

CREATE ROLE artifactory
WITH
  LOGIN
  PASSWORD 'avici'
  NOSUPERUSER
  NOCREATEDB
  NOCREATEROLE
  NOREPLICATION
  INHERIT;

# create database owned by this role

CREATE DATABASE artifactory
  OWNER artifactory
  ENCODING 'UTF8'
  LC_COLLATE 'en_US.UTF-8'
  LC_CTYPE 'en_US.UTF-8'
  TEMPLATE template0;

# restrict public access

REVOKE ALL ON DATABASE artifactory FROM PUBLIC;
GRANT ALL PRIVILEGES ON DATABASE artifactory TO artifactory;

# connect to new db

\c artifactory

# ensure schema ownership is correct

ALTER SCHEMA public OWNER TO artifactory;
GRANT ALL ON SCHEMA public TO artifactory;

# exit

\q

# exit postgres system user

exit

# verify connection on OS level

psql -h localhost -U artifactory -d artifactory

# edit jfrog system.yaml file 

cd /opt/jfrog/artifactory/var/etc/system.yaml

# create or open the yaml and put this inside, CAREFULL, yaml is space-sensitive

shared:
  database:
    type: postgresql
    driver: org.postgresql.Driver
    url: jdbc:postgresql://localhost:5432/artifactory
    username: artifactory
    password: avici

# set correct ownership

sudo chown artifactory:artifactory /opt/jfrog/artifactory/var/etc/system.yaml
sudo chmod 600 /opt/jfrog/artifactory/var/etc/system.yaml



# verify listener

psql -h localhost -U artifactory -d artifactory

# enable SCRAM-Authentication

vi /var/lib/pgsql/17/data/postgresql.conf

# set

password_encryption = scram-sha-256

# edit config

vi /var/lib/pgsql/17/data/pg_hba.conf

# will see this

local   all             all                                     scram-sha-256
host    all             all             127.0.0.1/32            scram-sha-256
host    all             all             ::1/128                 scram-sha-256

# add this to the top

local   all   postgres   peer

# reload config

J

# set password for postgres superuser

sudo -u postgres psql

ALTER USER postgres WITH PASSWORD 'avici';
\q

# remove the first line from the pg_hba.conf

vi /var/lib/pgsql/17/data/pg_hba.conf

# Delete:
# local   all   postgres   peer

systemctl reload postgresql-17

# verify login via scram

psql -U postgres -h 127.0.0.1 -d postgres

	# create artifactory db + role

	psql -U postgres -h 127.0.0.1 -d postgres

	*****CREATE ROLE artifactory WITH LOGIN PASSWORD 'STRONG_ARTIFACTORY_PASSWORD';
	*****CREATE DATABASE artifactory OWNER artifactory;


	\l
	\q

# verify artifactory db

psql -U artifactory -h 127.0.0.1 -d artifactory
\q

# install required extensions

sudo dnf install postgresql17-contrib

sudo -u postgres psql -d artifactory | sudo yum install postgresql17-contrib

# verifiy

ls /usr/pgsql-17/share/extension/pg_trgm.control

# creating the extension

CREATE EXTENSION IF NOT EXISTS pg_trgm;
\dx
\q

# install postgresql jdbc driver - system-wide

sudo -i

dnf -y install postgresql-jdbc

# Service state
systemctl status postgresql-17 --no-pager

# Port check
ss -lntp | grep 5432

# Confirm extension installed
sudo -u postgres psql -d artifactory -c "\dx" | grep pg_trgm

# Confirm contrib installed
rpm -qa | grep postgresql17-contrib

# Confirm version
psql -U postgres -h 127.0.0.1 -d postgres -c "SELECT version();"


```







```bash
# variant b
# switch to postgres system user

sudo -u postgres psql

# enter psql

psql

# inside psql, first cleanup, drop old ones if exists

DROP DATABASE IF EXISTS artifactory;
DROP ROLE IF EXISTS artifactory;

# create dedicated artifactory role

CREATE ROLE artifactory
WITH
  LOGIN
  PASSWORD 'StrongSecurePasswordHere'
  NOSUPERUSER
  NOCREATEDB
  NOCREATEROLE
  NOREPLICATION
  INHERIT;

# create database owned by this role

CREATE DATABASE artifactory
  OWNER artifactory
  ENCODING 'UTF8'
  LC_COLLATE 'en_US.UTF-8'
  LC_CTYPE 'en_US.UTF-8'
  TEMPLATE template0;

# restirct public access

REVOKE ALL ON DATABASE artifactory FROM PUBLIC;
GRANT ALL PRIVILEGES ON DATABASE artifactory TO artifactory;

# connect to database

\c artifactory

# ensure scheam ownership is correct

ALTER SCHEMA public OWNER TO artifactory;
GRANT ALL ON SCHEMA public TO artifactory;

# exit psql

\q

# exit postgresql system user

exit

# verify connection from OS level

psql -h localhost -U artifactory -d artifactory

```




### Point Artifactory to external Database

```bash
# go to 

$JFROG_HOME/artifactory/var/etc/system.yaml


shared:
  database:
    type: postgresql
    driver: org.postgresql.Driver
    url: jdbc:postgresql://<DB_SERVER_IP_OR_HOSTNAME>:5432/artifactory_db
    username: artifactory_user
    password: your_secure_password

# start artifactroy

sudo systemctl start artifactory.service




journalctl -xeu artifactory.service


artifactory-service.log
artifactory.log


| Item        | Location                          |
| ----------- | --------------------------------- |
| system.yaml | `/opt/jfrog/artifactory/var/etc/` |
| Logs        | `/opt/jfrog/artifactory/var/log/` |
| DB          | localhost:5432                    |
| Service     | systemd                           |

```


If admin / password does NOT work

That only happens if:

You restarted with an already initialized DB

Or the DB was partially initialized earlier

In that case, reset the admin password directly in PostgreSQL:

psql -h 127.0.0.1 -U artifactory -d artifactory


Then inside psql:

UPDATE users
SET password = '$2a$10$5Qb0x7vWQKQ4o3t5DqkT8uP8uYkVgJ0Qk2VqYkVgJ0Qk2VqYkVgJ0'
WHERE username = 'admin';


(That hash corresponds to password.)

Then restart:

sudo systemctl restart artifactory


But in 99% of clean installs:

👉 admin / password









