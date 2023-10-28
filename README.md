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

## Launch EC2 Ubuntu22 with T3.Medium and run a docker container for Sonarqube

## Install docker and start a container
	
 	sudo apt-get install docker.io -y
	sudo usermod -aG docker ubuntu
	sudo chmod 777 /var/run/docker.sock
	docker run -d --name sonarqube -p 9000:9000 -p 9092:9092 sonarqube
	netstat -tnpl
	docker ps -a

 ## Log in to sonarqube and create a token for a Jenkins

 By default, sonarqube

  	username - admin
   	password - admin

Then you have to reset the password for sonarqube

## Generate a Sonarqube token

Navigate to My Account -> Security -> Generate a new token

![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/78d8555b-15ea-4118-b2c2-70cceb709b4b)

## Configure sonarqube webhook for Quality Status Code check

Navigate to Administration -> Configuration -> Webhooks -> Create new

![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/b4109dcd-beb1-4549-9f31-2f53f9e80b89)

	Note: http://<Jenkins-Public-IP>:8080/sonarqube-webhook/
 
## Configure sonarqube token in Jenkins

Jenkins -> Manage Jenkins -> Credentials -> System -> Global credentials -> Add new

	Select - Secret Text
	Secret - Place your token
	ID - sonar-token
	Desrciption - sonar-token
 	Create

## Install sonarqube plugins in Jenkins

Installed plugins are listed below

	Sonarqube scanner for jenkins
	Sonarqube Generic Coverage Plugin
	Sonargraph Integration Jenkins plugins
	Sonar Quality Gates plugins
	Sonar Gerrit Plugin
	Quality Gates Plugin

 ## Configure Sonarqube installation on Jenkins Tools

 Jenkins -> Manage Jenkins -> Tools

	Select - Sonarqube Scanner Installations
	Name - sonarqube-scanner
	Select - Install automatically
	Version - SonarQube Scanner 5.0.1.3006

 ## Configure Sonarqube URL in Jenkins System

 Jenkins -> Manage Jenkins -> System

	Select - Sonarqube servers
	Name - sonarqube-server
	Server URL - http://<sonarqube-private-ip>:9000
	Server authentication token - <sonar-token> //which was created just before in Jenkins Global credentials
	Apply and save

## Update the Jenkinsfile for Sonarqube analysis and Quality status check 

	 https://github.com/kohlidevops/myregisterapp/blob/master/MyJenkinsFile

## Start the build

![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/15ea4fe7-4d2f-4e30-b285-0b8d831bf2f8)

## Build and Push Docker Image

Firstly install Docker related plugins in Jenkins. Installed plugins are listed below.

	Docker
	Docker Commons
	Docker Pipeline
	Docker API
	docker-build-step
	CloudBees Docker Build and Publish
	Docker Compose Build Step

## Generate Temporary Access Token for Jenkins to access

Navigate to hub.docker.com to login -> Select -> My account -> Security -> Generate Access Token

![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/cf29771f-93a3-4c5d-9a0b-375498a1d3b2)

## Configure Docker Access Token in Jenkins

Jenkins -> Manage Jenkins -> Credentials -> System -> Global Credentials -> Add new

	Select - Username with password
	username - latchudevops
	password - <Generated-Access-Token>
	ID - dockerhub
	Description - Dockerhub
	save

## Update the Jenkinsfile for Environment variables and Docker Build & Push

	https://github.com/kohlidevops/myregisterapp/blob/master/MyJenkinsFile

## Start the Jenkins Build

Perfect! My Jenkins job build has been succeded with complete CI.

![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/59ef5117-7e1e-426b-bdc2-e2275cdc7d57)


	
	
 



 
       
 	



