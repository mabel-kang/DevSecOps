# DevSecOps

## Contents
1. Overview
2. Prerequisites 
3. Setting up Jenkins


## Overview
The creation of this pipeline is done on Ubuntu 20.04 locally and the pipeline script can be found in the Jenkinsfile in this repository. 

## Prerequisites
#### 1. Install Java
```
sudo apt update
sudo apt install openjdk-11-jdk
```
Verify the installation:
```
java -version
```

#### 2. Install Git
```
sudo apt update
sudo apt install git
```
Verify the installation:
```
git --version
```

#### 3. Install Docker
i. Update the apt package index and install packages to allow apt to use a repository over HTTPS:
```
 sudo apt-get update
 sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```
ii. Add Docker’s official GPG key:
```
 curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
iii. Use the following command to set up the stable repository:
```
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
iv. Update the apt package index, and install the latest version of Docker Engine and containerd:
```
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```
v. Verify that Docker Engine is installed correctly by running the hello-world image:
```
 sudo docker run hello-world
```

#### 4. Install Jenkins
```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt-get update
sudo apt-get install jenkins
```
Register the Jenkins service:
```
sudo systemctl daemon-reload
```

Start the Jenkins service:
```
sudo systemctl start jenkins
```

Check the status of the Jenkins service:
```
sudo systemctl status jenkins
```
Jenkins can now be accessed at http://localhost:8080

Login to Jenkins by obtaining the password from:
```
sudo cat /var/lib/Jenkins/secrets/initialAdminPassword
```
This documentation is based on proceeding as admin, but there is the option of creating a new user for Jenkins. 

Install suggested plugins when prompted.

#### 5. Install SonarQube
```
docker run -d --name sonarqube -p 9000:9000 sonarqube
```
SonarQube can now be accessed at http://localhost:9000
Default username: admin, default password: admin 

## Create a pipeline with Jenkins

**Jenkins** -> **New Item** -> **Pipeline**

Under **Build Triggers**, 
   
   
Under **Pipeline**, 

Select **Definition** as **Pipeline script from SCM**







