pipeline {
   tools {
        maven 'Maven3'
    }
    
    agent any
    environment {     
            imagename = "abdulsukku/docker-new"
            registryCredential = 'dockerpass'
            dockerImage = ''    
    } 
   
    stages {
        stage('Cloning Git') {
            steps {
                    checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'githubcred', url: 'https://github.com/AbdulShukur007/myeksapp.git']]])
            }
        }
        stage('SonarQube analysis') {
            steps{
                 script{
                      withSonarQubeEnv('sonarserver') { 
                      sh "mvn sonar:sonar"
                      }
                  }
             }
        } 
        stage("Quality gate") {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }  
        stage ('Build') {
            steps {
                sh 'mvn clean install'           
            }
        }
        stage("Docker build"){
            steps{
                script {
                    dockerImage = docker.build imagename
                }
            }
        }
        stage("Push Image to Docker Hub"){
            steps{
                script {
                    docker.withRegistry( '', registryCredential ) {
                    dockerImage.push("$BUILD_NUMBER")
                    dockerImage.push('latest')
                    }
                }
            }
        }
        stage("kubernetes deployment"){
            steps{
                script {
                        withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'K8s', namespace: '', serverUrl: '') {
                        sh ('kubectl apply -f eks-deployment.yaml')
                    }
                } 
            }
        }
      
    }
}
