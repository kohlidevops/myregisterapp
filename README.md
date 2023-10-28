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

Perfect! My Jenkins job build has been succeded.

![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/59ef5117-7e1e-426b-bdc2-e2275cdc7d57)

## Update the Jenkinsfile for scanning Docker image and Cleanup

	https://github.com/kohlidevops/myregisterapp/blob/master/MyJenkinsFile

## Start the Jenkins Build again

Perfect! My Jenkins job build has been succeded. Now, Our Continuous Integration (CI) Job has been done.

![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/84ca2c1f-958b-45d9-a274-ad19b8cc382b)

## Set up EKSCTL Bootstrap server

## Install AWS Cli on the above EC2

	$ sudo su
	$ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
	$ apt install unzip,   $ unzip awscliv2.zip
	$ sudo ./aws/install

## Installing kubectl

	$ sudo su
 	$ cd ~
	$ curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.1/2023-04-19/bin/linux/amd64/kubectl
	$ chmod +x ./kubectl  
	$ mv kubectl /bin   
	$ kubectl version --output=yaml

## Set up Kubernetes using eksctl

	$ curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
	$ cd /tmp
	$ sudo mv /tmp/eksctl /bin
	$ eksctl version

## Assign IAM Role to eksctl instance

Create IAM Role for eksctl instance

	Trusted Entity - EC2
	Permission - Administrator Access //For demo only
	Create and assign this IAM role to eksctl instance

## Setup Kubernetes using eksctl

	$ cd ~
	$ eksctl create cluster --name virtualtechbox-cluster \
	--region ap-south-1 \
	--node-type t2.small \
	--nodes 3
	It will take some times to launch completely
	$ kubectl get nodes

![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/3e450154-d5b6-47d7-b485-808a5964804c)

## ArgoCD Installation on EKS Cluster and Add EKS Cluster to ArgoCD
### First, create a namespace
	$ kubectl create namespace argocd

### Next, let's apply the yaml configuration files for ArgoCd
	$ kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

### Now we can view the pods created in the ArgoCD namespace.
	$ kubectl get pods -n argocd

### To interact with the API Server we need to deploy the CLI:
	$ curl --silent --location -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/v2.4.7/argocd-linux-amd64
	$ chmod +x /usr/local/bin/argocd

### Expose argocd-server
	$ kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
	$ kubectl get svc -n argocd
 
#### Have to use the below loadbalancer URL to access the ArgoCD

![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/35c2c920-fb2f-46e7-b221-a3fbe1ad66e9)

## To decode the pasword
    $ kubectl get secret argocd-initial-admin-secret -n argocd -o yaml
    $ echo WXVpLUg2LWxoWjRkSHFmSA== | base64 --decode

![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/e61ff644-dee0-4bc4-8e7c-fb600108b763)

## To update the ArgoCD password

Select -> User Info -> To update the password

![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/e59b0aab-717d-4753-8d09-337d65b6f279)

## To Add EKS Cluster to ArgoCD

Firstly We have to login to ArgoCD from EKS Cluster through CLI

	$ argocd login a2255bb2bb33f438d9addf8840d294c5-785887595.ap-south-1.elb.amazonaws.com --username admin

 ![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/f7bd9860-210d-4ea7-9296-ac8fee3d1a63)

## To list the ArgoCD Cluster

	$ argocd cluster list

 As of now, we have default cluster, which is also shown in below image.

 ![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/7220120e-b389-472c-871c-2a512c29437b)

We have to add the EKS Cluster to the ArgoCD.

## To Show EKS Cluster

	$ kubectl config get-contexts

![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/dce961c4-d0c9-49ff-9e6b-b7ea7fd03149)

## To add the EKS Cluster to the ArgoCD using below command

	$ argocd cluster add i-0be779215a465a500@myapp-cluster.ap-south-1.eksctl.io --name myapp-eks-cluster

![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/a99d3336-28fe-4ea4-854b-2bf2d265bf27)

 Myapp-cluster has been added to ArgoCD Cluster.

 ![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/87525111-f4f3-4a90-a9bd-e31887864794)

### This is one of my repository which is having manifest file for kubernetes. Now I would like to connect this repository to the ArgoCD cluster.

 	https://github.com/kohlidevops/gitops-register-app

## Connect the repository from ArgoCD

Navigate to ArgoCD console -> Settings -> repository -> Add repository

	Connection method - Https
	Type - Git
	Project - Default
	Repository URL - https://github.com/kohlidevops/gitops-register-app
	Username - kohlidevops
	Password - <Generated-Personal-Access-Token-in-Jenkins>
	Then Connect

![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/dd3309cf-df0e-40c8-b6b6-c9e7e19dffc9)

Yes! Its connected successfully.

![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/e25ffced-7b92-483d-a97e-be42d569b403)

Just I had a look at the gitops repos for kubernetes to check the deployment-yaml file.

![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/952c4080-29e6-4279-a274-0d09f8abb0da)

Image ID and version should be match with the Docker hub.

![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/201ed418-0797-4b63-b4c8-dd2e20eab3ae)

## To add a app in ArgoCD

Navigate to ArgoCD console -> Application -> New app

	Application name - myapp
	Project name - default
	SYNC Policy - Automatic -> Select -> PRUNE Resources and SELF Heal
	Source - Repository URL - https://github.com/kohlidevops/gitops-register-app
	Revision - HEAD
	Path - ./ 
	Destination - <ArgoCD-Cluster-URL>
	Namespace - default
	Then Create

 Perfect! My Application status has been Passed.

 ![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/303cbf4a-b620-48db-afb6-57e26c40c9cd)

## If you execute the below command to see the number of Pods are running in the default namespace of my EKS Cluster

![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/cb6fad8b-329b-4f85-87d2-cc063e7c3b84)

## If you want to see the external DNS for default namespace

	$ kubectl get svc

![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/7b4d13cf-2bcb-4fff-ad53-edac9ded6392)

## As of now, Deployment has been successful on Kubernetes

![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/73c96656-6593-435c-8768-0cd7fb2ec55d)

We have to automate this process hereafter to complete the task!

## Create Jenkins API Token

Jenkins -> Select your Admin user -> Configure -> Choose -> API Token -> Add new Token

![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/4c47bc6b-bac2-4363-aa99-0001418758f2)

Note the Token ID

## Configure JENKINS_API_TOKEN in Jenkins credentials manager

Jenkins -> Manage Jenkins -> Credentials -> System -> Global credentials -> Secret Text

![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/5afbca70-af4b-4b9e-9a13-657359cff2d7)

Then add below variables in MyJenkinsFile Environment section

	JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
	https://github.com/kohlidevops/myregisterapp/blob/master/MyJenkinsFile

 ### Finally your ArgoCD stage should be updated

	 stage("Trigger ArgoCD Pipeline") {
	            steps {
	                script {
	                    sh "curl -v -k --user latchu:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-3-7-21-162.ap-south-1.compute.amazonaws.com:8080/job/gitops-register-app-cd/buildWithParameters?token=CD-Token'"
	                }
	            }
	       }
	Changes to be needed - Jenkins user, JENKINS_API_TOKEN, IMAGE_TAG, Jenkins Master EC2 DNS, and Token name
 
## Create a new job for Continuous Deployment

Jenkins -> New Item with Pipeline

	Select - This project is parameterized
	Name - IMAGE_TAG
	//Because Everytime my CD project will change the Image Tag in manifest file of Kubernetes repository in GitHub
	Build Triggers - Triggers build remotely
	Authentication Token - CD-Token
	Pipeline - Definition - Pipeline script from SCM
	SCM - Git
	Repository URL - https://github.com/kohlidevops/gitops-register-app
	Credentials - <Select>
	Branch specifier - */main
	Script path - Jenkinsfile

 I have update a Jenkinsfile for Kubernetes repository, which is going to perform Git Checkout, updating IMAGE_TAG in deployment yaml file and finally update this file to the kubernetes repository.

	https://github.com/kohlidevops/gitops-register-app/blob/main/Jenkinsfile

 The "IMAGE_TAG" value has been passed with the help of final stage of CI job and the Environment section. 

## Check and Validate the CICD

Do some changes in your application repository in GitHub and do manual build in CI job. I will not configure the GitSCM Polling in Jenkins CI job to Automate the CI Process once the Developer has been changed their code. (That's why Im doing manual build in CI job). I hope this GitSCM Polling and Jenkins webhook is not big deal for everyone.

### After the CI build, below things will happen

	CI Job - Workspace cleaning
	CI Job - Tool installation
	CI Job - GitHub Checkout (for myapp)
	CI Job - Build Application
	CI Job - Test Application
	CI Job - Sonarqube Analysis
	CI Job - Quality Gate Status
	CI Job - Build Docker Image using Dockerfile
	CI Job - Push Docker Image to DockerHub
	CI Job - Scanning Docker ImageHub Image
	CI Job - Triggering CD Job (Another Job for ArgoCD) - Here - I'm passing IMAGE_TAG, Jenkins admin username, ArgoCD job URL, JENKINS_API_TOKEN, and Jenkins Token for start build.

### After the CD build, below things will happen

	CD Job - Workspace cleaning
	CD Job - GitHub Checkout (for Kubernetes repository)
	CD Job - Docker Image Tag updation in Manifest Deployment Yaml file
	CD Job - Deployment changes updated to Kubernetes Repository
	CD Job - ArgoCD auto Pull the changes from repository
	CD Job - ArgoCD deploy the changes to the kubernetes cluster

### That's it.

If I start the CI build, then its successfully completed.

![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/db8fc798-a340-42c7-bc9b-e2d0252e257a)

This build trigger the CD Job that is ArgoCD Pipeline with passing some parameters

![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/5c2184cb-de76-4133-8717-3ca23fe022e8)

My Docker Image - That is version 1.0.0-15 has been updated in Docker Hub

![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/aa357729-8b45-49e9-86e8-115e68a4b092)

Same Image has been update in the manifest file (deployment.yml) of the kubernetes repository

![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/cc3f6653-6761-4720-9591-c1a64e823de2)

My ArgoCD application has been updated successfully

![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/c2383eae-e47d-4a5c-8846-ad7b254f2117)

If I refresh my application, then it should get updated.

![image](https://github.com/kohlidevops/myregisterapp/assets/100069489/72211d59-bb64-49e7-8a0f-cf36a2247b84)

## Cool!

