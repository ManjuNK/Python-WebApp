pipeline {
    agent any

    environment {
        SCANNER_HOME=tool 'SonarQube'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ManjuNK/Python-WebApp.git'
            }
        }
        
        stage("OWASP Dependency"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage("Trivy FS "){
            steps{
                sh "trivy fs ."
            }
        } 
        
        stage('code review'){
            steps{
                withSonarQubeEnv('Sonar-Server-9.4'){
                def mavenHome = tool name: "Maven-3.8.6", type:"maven"
                def mavenCMD = "${mavenHome}/bin/mvn"
                sh "${mavenCMD} sonar:sonar"
                }
        }
        
    } 
        
        stage("Docker Build and Tag  "){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh "make image"
                   }
                }
            }
        } 
        
         stage("Trivy Image Scan "){
            steps{
                sh "trivy image manjunk/python-webapp "
            }
        } 
        stage("Docker Push"){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh "make push"
                   }
                }
            }
        } 
        
        stage("Deploy to container"){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh "docker run -d -p 5000:5000 manjunk/python-webapp:latest "
                   }
                }
            }
        } 
    }
}
