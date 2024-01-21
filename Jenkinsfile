pipeline {
    agent any
    
    tools{
        maven 'maven3'
        jdk 'jdk17'
    }
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/mtosin123/CICD-Devops.git'
            }
        }
    
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Unit Tests') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=CICD-Devops -Dsonar.projectName=CICD-Devops \
                    -Dsonar.java.binaries=. '''
                }
            }
        }
        
        stage('OWASP Dependency Check') {
            steps{
                dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Build') {
            steps{
                sh "mvn package -DskipTests=true"
            }
        }
        
        stage('Deploy to Nexus') {
            steps{
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests=true"

                }
            }
        }
        
        stage('Docker Build and tag Image') {
            steps{
                script{
                 withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                     sh "docker build -t mtosin23/tars-app:latest -f docker/Dockerfile ."
                    }   
                }
            }
        }
        stage('Trivy Scan') {
            steps{
                sh "trivy image adijaiswal/ekart:latest > trivy-report.txt "
                }
            }
        stage('Docker Push Image') {
            steps{
                script{
                 withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                     sh "docker push adijaiswal/ekart:latest"
                    }
                }
            }
        }
        stage('Kubernetes Deploy') {
            steps{
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'K8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.89.141:6443') {
                sh "kubectl apply -f deploymentservice.yml -n webapps"
                sh "kubectl get svc -n webapps"
                    
                }
            }
        }
    }
}
    
    
