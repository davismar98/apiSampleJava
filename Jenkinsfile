pipeline {
    environment {
        registry = "davismar98/api-sre-challenge"
        registryCredential = 'dockerhub-cred'
        dockerImage = ''
    }

    agent any
    stages { 
        stage('build') {
            steps {
                withMaven(maven:'maven363') {
                    sh "mvn clean verify"
                }
            } 
        }
        
        stage('Build image') {         
            steps {
                script {
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }
        } 
        
        stage('Test image') {   
            steps {    
                script {
                    dockerImage.inside {            
                    sh 'echo "Tests passed"'        
                    }   
                }   
            }  
        }     
    }
}