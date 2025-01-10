pipeline {
    agent any
    tools {
        maven 'maven3'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'start', url: 'https://github.com/chetanKnayak/Multi-Tier-With-Database.git'
            }
        }

        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Tests') {
            steps {
                sh "mvn test"
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --format table -o fs.html ."
            }
        }

        stage('SonarQube') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=bankapp -Dsonar.projectName=bankapp \
                        -Dsonar.java.binaries=target'''
                }
            }
        }

        stage('Build and Publish to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'devopsshack-settings', maven: 'maven3') {
                    sh "mvn deploy"
                }
            }
        }

        stage('Build & tag Docker image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker cred') {
                        sh "docker build -t ckn97/bankapp:latest ."
                    }
                }
            }
        }

        stage('Trivy image Scan') {
            steps {
                sh "trivy image --format table -o image.html ckn97/banlapp:latest"
            }
        }

        stage('Publish Docker image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker cred') {
                        sh "docker push ckn97/bankapp:latest"
                    }
                }
            }
        }
        stage('Kubernetes Deploy') {
    steps {
        withKubeConfig(
            caCertificate: '', 
            clusterName: 'devopsshack-cluster', 
            contextName: '', 
            credentialsId: 'k8-token', 
            namespace: 'webapps', 
            restrictKubeConfigAccess: false, 
            serverUrl: 'https://DE5E6D8FC9CC242B10665F87422A7A47.gr7.us-east-1.eks.amazonaws.com'
        ) {
            sh "kubectl apply -f ds.yaml -n webapps"
            sleep 30
        }
    }
}

stage('Verify Deploy') {
    steps {
        withKubeConfig(
            caCertificate: '', 
            clusterName: 'devopsshack-cluster', 
            contextName: '', 
            credentialsId: 'k8-token', 
            namespace: 'webapps', 
            restrictKubeConfigAccess: false, 
            serverUrl: 'https://DE5E6D8FC9CC242B10665F87422A7A47.gr7.us-east-1.eks.amazonaws.com'
        ) {
            sh "kubectl get svc -n webapps"
            sleep 30
        }
    }
}

        }
    }
