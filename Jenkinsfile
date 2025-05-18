pipeline {
    agent any
    tools{
        maven 'maven'
    }
    environment {
        SCANNER_HOME= tool 'sonarqube'
        IMAGE_TAG = "v${BUILD_NUMBER}"
    }
    stages {
        stage('checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Mahi-tes/bankapp.git'
            }
        }
        
        stage('compile') {
            steps {
                sh 'mvn compile'
            }
        }
        
        stage('test') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('trivy fs scan') {
            steps {
                sh 'trivy fs --format table -o trivy-report.html .'
            }
        }
        
        stage('sonarqube analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=bank -Dsonar.projectName=bank \
                        -Dsonar.java.binaries=target'''
                     }
            }
        }
        
        stage('quality gate check') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                   waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                 }
            }
        }
        
        stage('build') {
            steps {
                sh 'mvn package'
            }
        }
        
        stage('artifact publish to nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'nexus-config', jdk: '', maven: 'maven', mavenSettingsConfig: '', traceability: true) {
                  sh 'mvn deploy'
                 }
            }
        }
        
        stage('docker build') {
            steps {
                script{
                withDockerRegistry(credentialsId: 'docker-cred') {
                  sh 'docker build -t 1mahendra/bankapp:$IMAGE_TAG .'
                    }
                }
            }
            
        }
        
        stage('trivy image scan') {
            steps {
                sh 'trivy image  --format table -o report.html 1mahendra/bankapp:$IMAGE_TAG'
            }
        }
        
        stage('push image to registery') {
            steps {
               script{
                withDockerRegistry(credentialsId: 'docker-cred') {
                  sh 'docker push 1mahendra/bankapp:$IMAGE_TAG'
                    }
                }
            }
        }
    }
}
