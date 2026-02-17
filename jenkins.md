

# Jenkins install guide (RHEL-Rocky 9.7)

## Update system

```bash
# make actual user sudo

su -
visudo
jenkins ALL=(ALL) NOPASSWD:ALL

sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/rpm-stable/jenkins.repo
sudo yum upgrade
# Add required dependencies for the jenkins package
sudo yum install fontconfig java-21-openjdk
sudo yum install jenkins
sudo systemctl daemon-reload


sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins

# copy initial password from /var/jenkins_home/secrets/initialAdminPassword
# in browser going to http://localhost:8080, and enter the initial admin password


```
