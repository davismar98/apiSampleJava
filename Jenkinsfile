pipeline {
    environment {\
        projectName = "api-sre-challenge"
        applicationName = "SRE Challenge API"
        registry = "davismar98/api-sre-challenge"
        registryCredential = 'dockerhub-cred'
        dockerImage = ''
        dockerTag = GIT_COMMIT.substring(0,7)
    }

    agent any

    triggers {
        pollSCM "* * * * *"
    }

    stages { 
        stage('Compile Application') {
            steps {
                withMaven(maven:'maven363') {
                    echo "*** Compiling ${applicationName} Application ***"
                    sh "mvn clean verify -DskipTests"
                }
            } 
        }

        stage('Test Application') {
            steps {
                withMaven(maven:'maven363') {
                    echo "*** Testing ${applicationName} Application ***"
                    sh "mvn test"
                }
            }
        }

        stage('Build Docker Image') {         
            steps {
                script {
                    echo "*** Building ${applicationName} Docker Image ***"
                    dockerImage = docker.build registry + ":$dockerTag"
                }
            }
        } 
        
        stage('Test Docker Image') {   
            steps {    
                script {
                    dockerImage.inside {  
                        echo "*** Testing ${applicationName} Docker Image ***"         
                        echo "[placeholder] Tests passed!"       
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
                        echo "*** Pushing ${applicationName} Docker Image ***" 
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Remove local Docker images') {
            steps {
                echo "*** Deleting ${applicationName} Local Docker Images ***" 
                sh ("docker rmi -f $registry:$dockerTag || :")
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
                        deployApp('stage')
                    }
                }  

                stage('PROD') {
                    when {
                        branch 'master' 
                    }
                    steps {
                        sh 'echo "[placeholder] Deploying to Production env..."'
                        deployApp('production')
                    }
                }  
            }
        }
    }
}

def deployApp(env) {
    sh "sed -i 's/%SRE_PROJECT_NAME%/${projectName}/g' kubernetes/deployment.yml kubernetes/service.yml"
    sh "sed -i 's/%IMAGE_VERSION%/${dockerTag}/g' kubernetes/deployment.yml kubernetes/service.yml"
    sh "cat kubernetes/deployment.yml"
    sh "cat kubernetes/service.yml"
}