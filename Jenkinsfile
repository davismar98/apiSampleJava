pipeline {
    environment {
        projectName = "api-sre-challenge"
        applicationName = "SRE Challenge API"
        registry = "davismar98/api-sre-challenge"
        registryCredential = 'dockerhub-cred'
        dockerImage = ''
        dockerTag = GIT_COMMIT.substring(0,7)
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
                        echo "*** Deploying ${applicationName} to DEVELOP environment ***" 
                        deployApp('develop')
                    }
                }

                stage('STAGE') {
                    when {
                        branch 'release' 
                    }
                    steps {
                        echo "*** Deploying ${applicationName} to STAGE environment ***" 
                        deployApp('stage')
                    }
                }  

                stage('PROD') {
                    when {
                        branch 'master' 
                    }
                    steps {
                        echo "*** Deploying ${applicationName} to PRODUCTION environment ***" 
                        deployApp('production')
                    }
                }  
            }
        }
    }
}

def deployApp(env) {
    echo '*** Rendering manifest for deployment ***'
    sh "sed -i 's/%SRE_PROJECT_NAME%/${projectName}/g' kubernetes/*.yml"
    sh "sed -i 's/%IMAGE_VERSION%/${dockerTag}/g' kubernetes/*.yml"
    
    withAWS(credentials:'aws-cred') {
        echo "*** Authenticating with the AWS EKS Cluster ***"
        sh "aws eks --region ${eks_cluster_region} update-kubeconfig --name ${eks_cluster_name}"
        
        echo "*** Updating Resources in namespace '${env}' ***"
        sh "kubectl apply -f kubernetes/ -n ${env}"
    }
}