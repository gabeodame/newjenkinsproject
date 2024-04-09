pipeline {
    tools {
        maven 'maven'
    }
    agent any
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Adeoriokin/newjenkinsproject.git']]])
            }
        }
        stage('Build Jar') {
            steps {
                def mvnHome = tool name: 'maven', type: 'maven'
                def mvnCMD = "${mvnHome}/bin/mvn "
                sh "${mvnCMD} clean package"
            }
        }
        stage('Docker Image Build') {
            steps {
                sh 'docker build -t projectjenkins .'
            }
        }
        stage('Push Docker Image to ECR') {
            steps {
                withAWS(credentials: 'aws-credentials', region: 'us-east-1') {
                    sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 730335395361.dkr.ecr.us-east-1.amazonaws.com'
                    sh 'docker tag projectjenkins:latest 730335395361.dkr.ecr.us-east-1.amazonaws.com/projectjenkins:latest'
                    sh 'docker push 730335395361.dkr.ecr.us-east-1.amazonaws.com/projectjenkins:latest'
                }
            }
        }
        stage('Integrate Jenkins with EKS Cluster and Deploy App') {
            steps {
                withAWS(credentials: 'aws-credentials', region: 'us-east-1') {
                  script {
                    sh ('aws eks --region us-east-1 update-kubeconfig --name Kubernetesproject')
                    sh '/var/lib/jenkins/kubectl apply -f eks-deploy-k8s.yaml'
                }
                }
        }
    }
    }
}
