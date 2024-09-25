pipeline {
    agent any
    tools {
        maven "maven3"
    }
    environment {
        SCANNER_HOME = tool "sonar-scanner"
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ganeshsnp987/Multi-Tier-With-Database.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test -DskipTests=true'
            }
        }
        
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs . --format table -o trivyfs.html"
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner \
                         -Dsonar.projectName=Multitier \
                         -Dsonar.projectKey=Multitier \
                         -Dsonar.java.binaries=target'''
                }
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        
        stage('Publish to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'settings-maven', maven: 'maven3', traceability: true) {
                    sh "mvn deploy -X -DskipTests=true"
                }
            }
        }
        
        stage('Docker build and tag Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker_cred') {
                        sh "docker build -t ganeshsnp987/bankapp:latest ."
                    }
                }
            }
        }
        
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image ganeshsnp987/bankapp:latest --format table -o trivyimage.html"
            }
        }
        
        stage('Docker Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker_cred') {
                        sh "docker push ganeshsnp987/bankapp:latest"
                    }
                }
            }
        }
        
        stage('Deploy to K8s') {
            steps {
                withKubeConfig(credentialsId: 'k8s', namespace: 'webapps', serverUrl: 'https://C5F50CA46FC28780AA4CBFC0BFE5E3EB.gr7.ap-south-1.eks.amazonaws.com') {
                    sh "kubectl apply -f ds.yml -n webapps"
                    sleep 30
                }
            }
        }
        
        stage('Verify K8s Deployment') {
            steps {
                withKubeConfig(credentialsId: 'k8s', namespace: 'webapps', serverUrl: 'https://C5F50CA46FC28780AA4CBFC0BFE5E3EB.gr7.ap-south-1.eks.amazonaws.com') {
                    sh "kubectl get pods -n webapps"
                    sh "kubectl get svc -n webapps"
                }
            }
        }
    }
}
