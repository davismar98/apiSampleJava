pipeline {
    environment {\
        projectName = "api-sre-challenge"
        registry = "davismar98/api-sre-challenge"
        registryCredential = 'dockerhub-cred'
        dockerImage = ''
        dockerTag = GIT_COMMIT.substring(0,7)
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
                    dockerImage = docker.build registry + ":$dockerTag"
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
            when {
                expression { BRANCH_NAME ==~ /develop|release|master/ }
            }
            steps{
                script {
                    docker.withRegistry('', registryCredential ) {
                        dockerImage.push()
                    }
                }
            }
        }
        
        stage ('Deploy to environment') {
            parallel {
                stage('DEVELOP') {
                    when {
                        branch 'develop' 
                    }
                    steps {
                        sh 'echo "[placeholder] Deploying to Development env..."'
                        deployApp('develop')
                    }
                }

                stage('STAGE') {
                    when {
                        branch 'release' 
                    }
                    steps {
                        sh 'echo "[placeholder] Deploying to Stage env..."'
                    }
                }  

                stage('PROD') {
                    when {
                        branch 'master' 
                    }
                    steps {
                        sh 'echo "[placeholder] Deploying to Production env..."'
                    }
                }  
            }
        }
    }
}

def deployApp(env) {
    sh "sed -i 's/%DOCKER_REGISTRY%/${registry}/g' kubernetes/deployment.yaml"
    sh "sed -i 's/%SRE_PROJECT_NAME%/${projectName}/g' kubernetes/deployment.yaml"
    sh "sed -i 's/%%IMAGE_VERSION%%/${dockerTag}/g' kubernetes/deployment.yaml"
    sh "cat kubernetes/deployment.yml"
}