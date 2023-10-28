Launch EC2 Ubuntu22 with T2.Micro for Jenkins Master

    ## Install Java
    $ sudo apt update
    $ sudo apt upgrade
    $ sudo hostnamectl hostname jenkins-master
    $ /bin/bash
    $ sudo apt install openjdk-17-jre
    $ java -version
    
    ## Install Jenkins
    Refer--https://www.jenkins.io/doc/book/installing/linux/
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

Launch EC2 Ubuntu22 with T2.Micro for Jenkins Agent

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

To unlock Jenkins and create one admin user

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

Install plugins in Jenkins

 	Maven Integration
  	Pipeline Maven Integration
   	Eclipse Temurin installer

Configure Java and Maven in Tools
	
	JDK Installations
	Name -  Java17
	Install automatically - Install from adoptium.net - Version - jdk-17.0.5+8

	Maven Installations
	Name - Maven3
	Install automatically - Version - 3.9.4

Create a Personal Access Token in GitHub that is your source code repository

 	Settings - Developer settings - Tokens(Classic) - Generate new token
  	Copy the Token to configure GitHub credentials in next step.

Configure GitHub credentials in Jenkins

	Jenkins - Manage Jenkins - Credentials - System - Global credentials
	Add - Username with password
	username - kohlidevops
	password - GitHub Personal Access Token - which was created in last step
	ID - github
	create a credential

 Create a GitHub repo with below sample app

	https://github.com/kohlidevops/myregisterapp
	Create a MyJenkinsFile for GitHub checkout, Maven Build and Test and save it.

 Create a new job with pipeline for my-app

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


 
       
 	



