pipeline {
    agent any
    stages {
        stage('build') {
            steps {
                withMaven(maven:'maven363') {
                    sh "mvn clean verify"
                }
            } 
        }
    }
}