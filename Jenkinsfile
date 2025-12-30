pipeline {
    agent any
    
    tools {
        maven 'maven3'
    }
    
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout ') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/sachi3050/Task-Master-Pro.git'
            }
        }
        
        stage('Code Compile') {
            steps {
                    sh "mvn compile"
            }
        }
        
        stage('Run Test Cases') {
            steps {
                    sh "mvn test"
            }
        }
        
        stage('Sonarqube Analysis') {
            steps {
                    withSonarQubeEnv('SonarQUBE') {
                        sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=TaskMasterPro-K8s \
                        -Dsonar.java.binaries=. \
                        -Dsonar.projectKey=TaskMasterPro-K8s '''
    
                }
            }
        }
        
        stage('OWASP Dependency Check') {
            steps {
                   dependencyCheck additionalArguments: '--scan ./   ', odcInstallation: 'ODC'
                   dependencyCheckPublisher pattern: '**/pom.xml'
            }
        }
        
        stage('Maven Build') {
            steps {
                    sh "mvn clean package"
            }
        }
        stage('Public to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'Global-Config-File', maven: 'maven3', traceability: true) {
                    sh 'mvn deploy'
                }
            }
        }
        
        stage('Docker Build & Tag') {
            steps {
                   script {
                       withDockerRegistry(credentialsId: 'Docker-Cred') {
                            sh "docker build -t taskpro ."
                            sh "docker tag taskpro sachidananda06/task-master:pro1"
                        }
                   } 
            }
        }
        
        stage('Docker Image scan') {
            steps {
                    sh "trivy image --format table -o image-report.html sachidananda06/task-master:pro1 "
            }
        }
        stage('Docker Push') {
            steps {
                   script {
                       withDockerRegistry(credentialsId: 'Docker-Cred') {
                            sh "docker push sachidananda06/task-master:pro1 "
                        }
                   } 
            }
        }
        stage('Deploy To K8s') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'sachi-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'task-master', restrictKubeConfigAccess: false, serverUrl: 'https://90501CC34D0A660C7714A7502FFC100E.gr7.us-east-1.eks.amazonaws.com') {
                    sh 'kubectl apply -f deployment-service.yml -n task-master'
                    sleep 30
                }
            }
        }
        stage('Verify Deployments') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'sachi-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'task-master', restrictKubeConfigAccess: false, serverUrl: 'https://90501CC34D0A660C7714A7502FFC100E.gr7.us-east-1.eks.amazonaws.com') {
                    sh 'kubectl get pods -n task-master'
                    sh 'kubectl get svc -n task-master'
                }
            }
        }
        
    }
}
