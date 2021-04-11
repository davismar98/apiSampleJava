pipeline {
    environment {
        projectName = "api-sre-challenge"
        applicationName = "SRE Challenge API"
        dockerRegistry = "davismar98/api-sre-challenge"
        dockerRegistryCredential = "dockerhub-cred"
        dockerImage = ""
        dockerTag = GIT_COMMIT.substring(0,7)
        env = ""
        eks_cluster_name = 'sre-challenge'
        eks_cluster_region = 'us-east-1'
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

        stage('Unit Test Application') {
            steps {
                withMaven(maven:'maven363') {
                    echo "*** Testing ${applicationName} Application ***"
                    sh "mvn test"
                }
            }
        }

        stage('Code Quality Analysis') {
            steps {
                echo "[placeholder] Code quality analysis integration (i.e SonarQube)"
            }
        }
        
        stage('Application Security Test (SAST, SCA)') {
            steps {
                echo "[placeholder] Software security test or Software Composition Analysis integration (i.e Veracode, Checkmarkx, Black Duck)"
            }
        }

        stage('Build Docker Image') {         
            steps {
                script {
                    echo "*** Building ${applicationName} Docker Image ***"
                    dockerImage = docker.build dockerRegistry + ":$dockerTag"
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
                    docker.withRegistry('', dockerRegistryCredential ) {
                        echo "*** Pushing ${applicationName} Docker Image ***" 
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Remove local Docker images') {
            steps {
                echo "*** Deleting ${applicationName} Local Docker Images ***" 
                sh ("docker rmi -f $dockerRegistry:$dockerTag || :")
            }
        }
        
        stage ('Deploy to environment') {
            parallel {
                stage('DEVELOP') {
                    when {
                        branch 'develop' 
                    }
                    steps {
                        script {
                            echo "*** Deploying ${applicationName} to DEVELOP environment ***" 
                            env = "develop"
                            deployApp("develop")
                        }
                    }
                }

                stage('STAGE') {
                    when {
                        branch 'release' 
                    }
                    steps {
                        script {
                            echo "*** Deploying ${applicationName} to STAGE environment ***" 
                            env = "stage"
                            deployApp(env)
                        }
                    }
                }  

                stage('PROD') {
                    when {
                        branch 'master' 
                    }
                    steps {
                        script { 
                            echo "*** Deploying ${applicationName} to PRODUCTION environment ***" 
                            env = "production"
                            deployApp(env)
                        }
                    }
                }  
            }
        }
    }

    post { 
        success { 
            script {
                if ( BRANCH_NAME ==~ /develop|release|master/ ) {
                    echo "*** Notifiying success result to Slack ***" 
                    slackSend color: "good", message: "Application ${applicationName} (version ${dockerTag}) has been successfully deployed to ${env}! Check ${BUILD_URL} for details."
                }
                    
            }
        }

        failure {
            script {
                if ( BRANCH_NAME ==~ /develop|release|master/ ) {
                    echo "*** Notifiying failed result to Slack ***" 
                    slackSend color: "danger", message: "Application ${applicationName} (version ${dockerTag}) could not be deployed to ${env}! Check ${BUILD_URL} for details."
                }
            }
        }
    }
}

def deployApp(eks_namespace) {
    echo '*** Rendering manifest for deployment ***'
    sh "sed -i 's/%SRE_PROJECT_NAME%/${projectName}/g' kubernetes/*.yml"
    sh "sed -i 's/%IMAGE_VERSION%/${dockerTag}/g' kubernetes/*.yml"
    
    withAWS(credentials:'aws-cred') {
        echo "*** Authenticating with the AWS EKS Cluster ***"
        sh "aws eks --region ${eks_cluster_region} update-kubeconfig --name ${eks_cluster_name}"
        
        echo "*** Updating Resources in namespace '${eks_namespace}' ***"
        sh "kubectl apply -f kubernetes/ -n ${eks_namespace}"
    }
}