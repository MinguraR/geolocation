pipeline {
    agent any
    tools{
        maven 'M2_HOME'
    }
     environment {
    registry = '598296426280.dkr.ecr.ap-south-1.amazonaws.com/geolocation_ecr_rep'
    registryCredential = 'jenkins-ecr'
    dockerimage = ''
  }
    stages {
        stage('Checkout'){
            steps{
                git branch: 'main', url: 'https://github.com/MinguraR/geolocation.git'
            }
        }
        stage('Code Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
          // Building Docker images
        stage('Building image') {
            steps{
                script {
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }
        // Uploading Docker images into AWS ECR
        stage('Pushing to ECR') {
            steps{
                script {
                    sh 'aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 598296426280.dkr.ecr.ap-south-1.amazonaws.com/geolocation_ecr_rep'
                    sh 'docker push 598296426280.dkr.ecr.ap-south-1.amazonaws.com/geolocation_ecr_rep:$BUILD_NUMBER'
                }
            }
        }
        //deploy the image that is in ECR to our EKS cluster
        stage ("Kube Deploy") {
            steps {
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'secret', namespace: '', serverUrl: '') {
                 sh "kubectl apply -f eks_deploy_from_ecr.yaml"
                }
            }
        }
    }
}
