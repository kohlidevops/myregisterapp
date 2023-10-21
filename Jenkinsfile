pipeline {
    agent { label 'jenkins-agent' }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    stages{
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
        }

        stage("Checkout from SCM"){
                steps {
                    git branch: 'master', credentialsId: 'github-login', url: 'https://github.com/kohlidevops/myregisterapp'
                }
        }

        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }
       }
       stage("Test Application"){
           steps {
                 sh "mvn test"
           }
       }
    }
}
    
