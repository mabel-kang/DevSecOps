# DevSecOps

## Contents
1. Overview
2. Install Tools
3. Pipeline Stages
4. Creating the pipeline in Jenkins


## Overview
The creation of this pipeline is done on Ubuntu 20.04 locally and the pipeline script can be found in the Jenkinsfile in this repository. 

## Install Tools
#### 1. Java
```
sudo apt update
sudo apt install openjdk-11-jdk
```
Verify the installation:
```
java -version
```

#### 2. Git
```
sudo apt update
sudo apt install git
```
Verify the installation:
```
git --version
```

#### 3. Docker
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

#### 4. Jenkins
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
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
This documentation is based on proceeding as admin, but there is the option of creating a new user for Jenkins. 

Install suggested plugins when prompted.

#### 5. SonarQube
```
docker run -d --name sonarqube -p 9000:9000 sonarqube
```
SonarQube can now be accessed at http://localhost:9000
Default username: admin, default password: admin 

## Pipeline Stages (based on the Jenkinsfile in this repository)
#### 1. Git-Checkout

```
steps {
    echo 'Checking out from Git Repo';
    git branch: 'master', url: 'https://github.com/mabel-kang/ip.git'
}
```
Checks out the github repository that is to be used in the pipeline. The repository used in this tutorial is source code for a JAR application which uses Gradle.

Prerequisites:
Ensure that the Git Plugin is installed. Otherwise go to **Manage Jenkins** -> **Manage Plugins** and install the Git Plugin.

#### 2. Check Git Secrets

```
steps {
    sh 'rm trufflehog || true'
    sh 'docker run -t gesellix/trufflehog --json https://github.com/mabel-kang/ip.git > trufflehog'
    sh 'cat trufflehog'
}
```

`rm trufflehog || true`: removes any file named **trufflehog** if it exists.   
`docker run -t gesellix/trufflehog --json https://github.com/mabel-kang/ip.git > trufflehog`: runs the trufflehog program in the docker container to check for git secrets in in the repository (https://github.com/mabel-kang/ip.git). Output will be stored in a file named **trufflehog**.   
`cat trufflehog`: the result of running the program will be shown in **Console Output**. 

To access **Console Output**: 
In **Dashboard**, select your pipeline. Under **Build History**, select the build you want to view, then select **Console Output**. 

Prerequisites:
Ensure that **jenkins** user is added to the **docker** group in your system so that the docker command can execute
```
sudo usermod -aG docker jenkins
```

#### 3. SAST

```
steps {
    withSonarQubeEnv('sonar') {
            sh './gradlew sonarqube'
    }
}
```
This execute the SonarQube analysis via a regular Gradle task. After the analysis is completed, view the results at http://localhost:9000. 

Prerequisites:
- Go to **Manage Jenkins** -> **Manage Plugins** -> **Available** and install the **SonarQube Scanner For Jenkins** plugin.
- Configuration:
  - Go to **Manage Jenkins** -> **Configure System**. Under the **SonarQube servers** section:
     - Select the **Environment variables** option
     - Click **Add SonarQube**, and add the values you're prompted for.
       The server authentication token should be created as a 'Secret Text' credential.
       Generate your token by going to **User** -> **My Account** > **Security**.
   - Go to **Manage Jenkins** -> **Global Tool Configuration**. Under the **SonarQube Scanner** section, click **Add SonarQube Scanner** and add the values you're prompted        for.
- In your **gradle.properties** file, include:
```
systemProp.sonar.host.url=http://localhost:9000
```
- In your **build.gradle** file, include: 
```
plugins {
  id "org.sonarqube" version "3.3"
}
```
#### 4. OWASP
```
steps {
    dependencyCheck additionalArguments: 'scan="src" --format HTML', odcInstallation: 'OWASP'
}
```
This attempts to detect publicly disclosed vulnerabilities contained within a project's dependencies. The report generated will be available in the **Workspace** of the build.

To access **Workspace**:
In **Dashboard**, select your pipeline. Under **Build History**, select the build you want to view, then select **Workspace**. 

Prerequisites: 
- Go to **Manage Jenkins** -> **Manage Plugins** -> **Available** and install the **OWASP Dependency-Check** plugin.
- Go to **Manage Jenkins** -> **Global Tool Configuration**. Under the **Dependency-Check** section, click **Add Dependency-Check** and add the values you're prompted        for.

#### 5. Run JUnit Tests
```
steps {
    sh './gradlew test'
}   
```
This runs all the JUnit tests in the repository. Results will be printed out in **Console Output**.

#### 6. Build JAR
```
steps {
    sh './gradlew shadowJar'
}   
```
This builds the JAR file from the source code.

## Creating the pipeline in Jenkins

**Jenkins** -> **New Item** -> Enter your item name and choose **Pipeline**

Under **Build Triggers**, select **Poll SCM**. Enter `* * * * *` to enable polling every minute so that build will automatically occur when changes in the repository is detected. 
   
Under **Pipeline**:
1. Select **Pipeline script from SCM** for **Definition**
2. Select **Git** for **SCM**
3. Enter the URL of your repository that contains your Jenkinsfile for **Repositories**. Add credentials if your repository is private.
4. Enter the correct branch specifier **Branches to build**
5. Enter `Jenkinsfile` as the **Script Path**

If you are using this repository, the pipeline section should look like the following:
![image](https://user-images.githubusercontent.com/65720353/125407765-c57ca780-e3ec-11eb-9989-a2b41a678309.png)

Choose the **Save** option after you have completed the configuration.  
Due to the polling, the pipeline will automatically build.  
You can view the progress under the **Stage View**.  






