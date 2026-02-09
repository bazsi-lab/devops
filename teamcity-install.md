

author:     saba    
version:    v1.0
date:       Januar 27th, 2026


# Teamcity install log

```bash
# System update and install base tools

sudo dnf update -y
sudo dnf install -y \
  curl \
  wget \
  tar \
  unzip \
  vim \
  git \
  firewalld

# install Extra Packages for Enterprise Linux - EPEL

sudo dnf install epel-release 

# install java openjdk  from manual tarball, correct release find under 
https://github.com/adoptium/temurin17-binaries/releases

wget https://github.com/adoptium/temurin17-binaries/releases/download/jdk-17.0.9+9/OpenJDK17U-jdk_x64_linux_hotspot_17.0.9_9.tar.gz

# extract the tarball

tar -xzf OpenJDK17U-jdk_x64_linux_hotspot_17.0.9_9.tar.gz

# Enable firewall (required for enterprise-style setups)
sudo systemctl enable --now firewalld

# move to directory

sudo mkdir -p /usr/lib/jvm
sudo mv jdk-17.0.9+9 /usr/lib/jvm/

# set environment variables

export JAVA_HOME=/usr/lib/jvm/jdk-17.0.9+9
export PATH=$JAVA_HOME/bin:$PATH

# apply changes

source /etc/profile

# verify installation

java -version

# Expected: openjdk version "17.x"

# ------------------------------------------
# create dedicated system user for teamcity
#-------------------------------------------
# best practice, never run cli services as root

sudo useradd \
  --system \
  --home /opt/teamcity \
  --shell /bin/bash \
  teamcity

#-------------------------------
# download and extract teamcity
#-------------------------------

cd /opt

# Download official TeamCity distribution (adjust version if needed)
sudo wget https://download.jetbrains.com/teamcity/TeamCity-2024.12.1.tar.gz

# Extract archive
sudo tar -xzf TeamCity-2024.12.1.tar.gz

# Ensure correct ownership
sudo chown -R teamcity:teamcity /opt/TeamCity

# ---------------------------------------------------------------------------
# create TeamCity data directory (separate from binaries)
# this holds configs, builds, logs, artifacts, DB files
# ---------------------------------------------------------------------------
sudo mkdir -p /var/lib/teamcity
sudo chown -R teamcity:teamcity /var/lib/teamcity


# ---------------------------------------------------------------------------
# define environment variables system-wide
# ---------------------------------------------------------------------------
sudo tee /etc/profile.d/teamcity.sh > /dev/null <<'EOF'
export TEAMCITY_HOME=/opt/TeamCity
export TEAMCITY_DATA_PATH=/var/lib/teamcity
EOF

# load variables into current shell
source /etc/profile.d/teamcity.sh

# ---------------------------------------------------------------------------
# create systemd service for TeamCity
# this allows controlled startup, restart, logging, monitoring
# ---------------------------------------------------------------------------
sudo tee /etc/systemd/system/teamcity.service > /dev/null <<'EOF'
[Unit]
Description=TeamCity Server
After=network.target

[Service]
Type=forking
User=teamcity
Group=teamcity
Environment=TEAMCITY_HOME=/opt/TeamCity
Environment=TEAMCITY_DATA_PATH=/var/lib/teamcity
ExecStart=/opt/TeamCity/bin/runAll.sh start
ExecStop=/opt/TeamCity/bin/runAll.sh stop
Restart=on-failure
RestartSec=10
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF

# reload systemd to apply new service
sudo systemctl daemon-reexec
sudo systemctl daemon-reload

# ---------------------------------------------------------------------------
# enable and start TeamCity
# ---------------------------------------------------------------------------
sudo systemctl enable --now teamcity

# check service status
systemctl status teamcity --no-pager


# ---------------------------------------------------------------------------
# open TeamCity web port (default: 8111)
# ---------------------------------------------------------------------------
sudo firewall-cmd --permanent --add-port=8111/tcp
sudo firewall-cmd --reload


# ---------------------------------------------------------------------------
# verify TeamCity is listening
# ---------------------------------------------------------------------------
ss -tulpen | grep 8111
# Expected: LISTEN on TCP port 8111


# ---------------------------------------------------------------------------
# logg inspection (important for first startup)
# ---------------------------------------------------------------------------
tail -f /opt/TeamCity/logs/teamcity-server.log
```

## Install Postgresql for Teamcity

```bash
# imortant: modules are decrepcated from rocky linux in the future, native install needed
# install dnf packages, no modules!

sudo dnf update -y
sudo dnf install -y \
  postgresql-server \
  postgresql-contrib

# verify

psql --version
# Expected: PostgreSQL 15.x (or newer, depending on Rocky updates)

# initialize postgresql - this creates /var/lib/pgsql/data

sudo postgresql-setup --initdb

# enable and start PostgreSQL service

sudo systemctl enable --now postgresql

# check status

systemctl status postgresql --no-pager

sudo sed -i \
  's/^host\s\+all\s\+all\s\+127.0.0.1\/32\s\+.*/host all all 127.0.0.1\/32 md5/' \
  /var/lib/pgsql/data/pg_hba.conf

sudo sed -i \
  's/^host\s\+all\s\+all\s\+::1\/128\s\+.*/host all all ::1\/128 md5/' \
  /var/lib/pgsql/data/pg_hba.conf


# ---------------------------------------------------------------------------
# ensure PostgreSQL listens only locally (security best practice)
# ---------------------------------------------------------------------------
sudo sed -i \
  "s/^#listen_addresses =.*/listen_addresses = 'localhost'/" \
  /var/lib/pgsql/data/postgresql.conf


# ---------------------------------------------------------------------------
# restart PostgreSQL to apply config
# ---------------------------------------------------------------------------
sudo systemctl restart postgresql


# 7) Create TeamCity database and user
# IMPORTANT:
# - UTF8 encoding REQUIRED
# - Collation must be C (TeamCity recommendation)
# ---------------------------------------------------------------------------
sudo -u postgres psql <<'EOF'
CREATE USER teamcity WITH PASSWORD 'STRONG_PASSWORD_HERE';
CREATE DATABASE teamcity
  OWNER teamcity
  ENCODING 'UTF8'
  LC_COLLATE 'C'
  LC_CTYPE 'C'
  TEMPLATE template0;
EOF

# NOTE:
# Replace STRONG_PASSWORD_HERE with a real password
# You will enter this later in TeamCity web setup


# ---------------------------------------------------------------------------
# 8) Validate database access
# ---------------------------------------------------------------------------
psql -h localhost -U teamcity -d teamcity -c '\conninfo'
# Expected: successful connection message


# ---------------------------------------------------------------------------
# 9) Firewall check
# PostgreSQL NOT exposed externally (intentional)
# TeamCity connects via localhost
# ---------------------------------------------------------------------------
sudo firewall-cmd --list-all
# No port 5432 required to be open


###############################################################################
# PostgreSQL is now READY for TeamCity
#
# Next step (Web UI):
# - Open TeamCity (http://<ip>:8111)
# - Select "PostgreSQL"
# - DB host: localhost
# - DB name: teamcity
# - User: teamcity
# - Password: <set above>
###############################################################################
```


on teamcity webui

- install postgresql database driver
- set the correct settings
- accept the license
- add an admin account >> censored


- create github account 
- fix from the vespa user the teamcity user's failing directoryes
- sudo mkdir -p /opt/teamcity
sudo chown teamcity:teamcity /opt/teamcity
sudo chmod 750 /opt/teamcity


- switch to teamcity account - 
  sudo -u teamcity -s
  whoami
  pwd

- generate ssh key and connect to github 
mkdir -p ~/.ssh
chmod 700 ~/.ssh

ssh-keygen -t ed25519 \
  -f ~/.ssh/github_ci \
  -C "teamcity@devops-ci-01"

chmod 600 ~/.ssh/github_ci
chmod 644 ~/.ssh/github_ci.pub


```

