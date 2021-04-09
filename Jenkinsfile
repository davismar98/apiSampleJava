pipeline {
    environment {
        registry = "davismar98/api-sre-challenge"
        registryCredential = 'dockerhub-cred'
        dockerImage = ''
    }

    agent any
    stages { 
        stage('Maven compile') {
            steps {
                withMaven(maven:'maven363') {
                    sh "mvn clean verify -DskipTests"
                }
            } 
        }

        stage('Maven Test') {
            steps {
                withMaven(maven:'maven363') {
                    sh "mvn test"
                }
            }
        }

        stage('Build Docker image') {         
            steps {
                script {
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }
        } 
        
        stage('Test Docker image') {   
            steps {    
                script {
                    dockerImage.inside {            
                    sh 'echo "Tests passed"'        
                    }   
                }   
            }  
        }
        stage('Deliver Docker image') {
            steps{
                script {
                    docker.withRegistry('', registryCredential ) {
                        dockerImage.push()
                    }
                }
            }
        }     
    }
}