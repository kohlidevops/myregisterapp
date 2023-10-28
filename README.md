## Launch EC2 Ubuntu22 with T2.Micro for Jenkins Master

## Install Java

    $ sudo apt update
    $ sudo apt upgrade
    $ sudo hostnamectl hostname jenkins-master
    $ /bin/bash
    $ sudo apt install openjdk-17-jre
    $ java -version
    
## Install Jenkins

    curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
      /usr/share/keyrings/jenkins-keyring.asc > /dev/null
    echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
      https://pkg.jenkins.io/debian binary/ | sudo tee \
      /etc/apt/sources.list.d/jenkins.list > /dev/null
    sudo apt-get update
    sudo apt-get install jenkins
    
    $ sudo systemctl enable jenkins       //Enable the Jenkins service to start at boot
    $ sudo systemctl start jenkins        //Start Jenkins as a service
    $ systemctl status jenkins
    $ sudo vi /etc/ssh/sshd_config

	    PubkeyAuthentication yes
	    AuthorizedKeysFile      .ssh/authorized_keys .ssh/authorized_keys2
   
    $ sudo systemctl reload sshd
    $ sudo service sshd reload
    $ ssh-keygen
    $ cat /home/ubuntu/.ssh/id_rsa.pub

Copy the id_rsa.pub key content and paste this content in /home/ubuntu/.ssh/authorized_keys in Jenkins Agent machine

## Launch EC2 Ubuntu22 with T2.Micro for Jenkins Agent

    $ sudo hostnamectl hostname jenkins-agent
    $ /bin/bash
    $ sudo apt-get update -y
    $ sudo apt-get upgrade -y
    $ sudo apt install openjdk-17-jre
    $ java --version
    $ sudo apt-get install docker.io -y
    $ sudo systemctl enable docker.service
    $ sudo systemctl status docker.service
    $ sudo usermod -aG docker $USER
    $ sudo vi /etc/ssh/sshd_config
    
    	PubkeyAuthentication yes
    	AuthorizedKeysFile      .ssh/authorized_keys .ssh/authorized_keys2
       
    $ sudo systemctl reload sshd
    $ sudo service sshd reload
    $ sudo vi /home/ubuntu/.ssh/authorized_keys
      Paste the id_rsa.pub key content which is copied in Jenkins Master.

## To unlock Jenkins and create one admin user

To make Jenkins Buildin Node as 0 and save it. Then create a new Node with Jenkins-Agent.

	Name - Jenkins-Agent
 	Number of executors - 2
  	Remote root directory - /home/ubuntu
   	Labels - Jenkins-Agent
    	Usage - Use this node as much as possible
     	Launch method - Launch agents via ssh
      	Host - Jenkins-Agent-Private-IP
       	Credentials - Add Jenkins - To add Username with SSH Private key
	Host key verification strategy - Non Verifying Verification Strategy
 	Availability - Keep this agent online as much as possible

save and check the Jenkins-Agent logs in Jenkins console - Should be connected.

To create a Test Jenkins pipeline with Hello world script to validate whether job is running without fail.

## Install plugins in Jenkins

 	Maven Integration
  	Pipeline Maven Integration
   	Eclipse Temurin installer

## Configure Java and Maven in Tools
	
	JDK Installations
	Name -  Java17
	Install automatically - Install from adoptium.net - Version - jdk-17.0.5+8

	Maven Installations
	Name - Maven3
	Install automatically - Version - 3.9.4

## Create a Personal Access Token in GitHub that is your source code repository

 	Settings - Developer settings - Tokens(Classic) - Generate new token
  	Copy the Token to configure GitHub credentials in next step.

## Configure GitHub credentials in Jenkins

	Jenkins - Manage Jenkins - Credentials - System - Global credentials
	Add - Username with password
	username - kohlidevops
	password - GitHub Personal Access Token - which was created in last step
	ID - github
	create a credential

 ## Create a GitHub repo with below sample app

	https://github.com/kohlidevops/myregisterapp
	Create a MyJenkinsFile for GitHub checkout, Maven Build and Test and save it.

 ## Create a new job with pipeline for my-app

 To open a job and select the Pipeline section

	Definition - Pipeline script from SCM
	SCM - Git
	Repository URL - https://github.com/kohlidevops/myregisterapp
	Credentilas - Select your credentials which was created in last step
	Branch specifier - */master
	Script path - MyJenkinsFile
	Apply - Save
	Test the Build

 Perfect! My build has been completed.

 ![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/43cdc823-1db5-411a-ac8d-5662d4a6f7e9)

Launch EC2 Ubuntu22 with T3.Medium and Install and configure Sonarqube

## Update Package Repository and Upgrade Packages
    $ sudo apt update
    $ sudo apt upgrade
## Add PostgresSQL repository
    $ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
    $ wget -qO- https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo tee /etc/apt/trusted.gpg.d/pgdg.asc &>/dev/null
## Install PostgreSQL
    $ sudo apt update
    $ sudo apt-get -y install postgresql postgresql-contrib
    $ sudo systemctl enable postgresql
## Create Database for Sonarqube
    $ sudo passwd postgres
    $ su - postgres
    $ createuser sonar
    $ psql 
    $ ALTER USER sonar WITH ENCRYPTED password 'sonar';
    $ CREATE DATABASE sonarqube OWNER sonar;
    $ grant all privileges on DATABASE sonarqube to sonar;
    $ \q
    $ exit
## Add Adoptium repository
    $ sudo bash
    $ wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
    $ echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
 ## Install Java 17
    $ apt update
    $ apt install temurin-17-jdk
 ## If any issues with above command then use below command to install temurin-17-jdk
    $ sudo wget https://packages.adoptium.net/artifactory/deb/pool/main/t/temurin-17/temurin-17-jdk_17.0.9.0+9_amd64.deb
    $ sudo dpkg -i temurin-17-jdk_17.0.9.0+9_amd64.deb
    $ update-alternatives --config java
    $ /usr/bin/java --version
    $ exit 
## Linux Kernel Tuning
   # Increase Limits
    $ sudo vim /etc/security/limits.conf
    //Paste the below values at the bottom of the file
    sonarqube   -   nofile   65536
    sonarqube   -   nproc    4096

    # Increase Mapped Memory Regions
    sudo vim /etc/sysctl.conf
    //Paste the below values at the bottom of the file
    vm.max_map_count = 262144

#### Sonarqube Installation ####
## Download and Extract
    $ sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.0.65466.zip
    $ sudo apt install unzip
    $ sudo unzip sonarqube-9.9.0.65466.zip -d /opt
    $ sudo mv /opt/sonarqube-9.9.0.65466 /opt/sonarqube
## Create user and set permissions
     $ sudo groupadd sonar
     $ sudo useradd -c "user to run SonarQube" -d /opt/sonarqube -g sonar sonar
     $ sudo chown sonar:sonar /opt/sonarqube -R
## Update Sonarqube properties with DB credentials
     $ sudo vim /opt/sonarqube/conf/sonar.properties
     //Find and replace the below values, you might need to add the sonar.jdbc.url
     sonar.jdbc.username=sonar
     sonar.jdbc.password=sonar
     sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
## Create service for Sonarqube
$ sudo vim /etc/systemd/system/sonar.service
//Paste the below into the file
     [Unit]
     Description=SonarQube service
     After=syslog.target network.target

     [Service]
     Type=forking

     ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
     ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

     User=sonar
     Group=sonar
     Restart=always

     LimitNOFILE=65536
     LimitNPROC=4096

     [Install]
     WantedBy=multi-user.target

## Start Sonarqube and Enable service
     $ sudo systemctl start sonar
     $ sudo systemctl enable sonar
     $ sudo systemctl status sonar

## Watch log files and monitor for startup
     $ sudo tail -f /opt/sonarqube/logs/sonar.log



 
       
 	



