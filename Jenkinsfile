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
        stage('Git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/d-abhi/Ekart.git'
            }
        }
        stage('compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Unit Test') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
        stage('SonarQube Anlysis') {
            steps {
                withSonarQubeEnv('sonar') {
                sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=EKART -Dsonar.projectName=EKART \
                -Dsonar.java.binaries=. '''
            
              }
            }
        }
        stage('OWASP Dependency Check') {
            steps {
              dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DC'
              dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
              
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        
        stage('upload artifacts to nexus') {
            steps {
               withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
               sh "mvn deploy -DskipTests=true"
               }
               
                
            }
            
        }
        
        stage('Build & Tag Docker image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker ') {
                        sh "docker build -t abhid92/ekart:latest -f docker/Dockerfile ."
                 }
            }
        }
		
		}
        
        stage('trivy Scan') {
            steps {
                sh "trivy image abhid92/ekart:latest > trivy-report.txt"
            }
        }
        
        stage('Push image to Docker Hub') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker ') {
                        sh "docker push abhid92/ekart:latest"
                 }
            }
        }
		
		}
        
        stage('kubernetes Deploy') {
            steps {
               withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://192.168.56.41:6443') {
                 sh "kubectl apply -f deploymentservice.yml -n webapps"
                 sh "kubectl get svc -n webapps"
               }
            }
        }
        
    
          
          }

      }
   
      



