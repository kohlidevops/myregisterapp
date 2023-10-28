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



