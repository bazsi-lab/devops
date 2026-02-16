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



#### Configre Postgresql

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

















