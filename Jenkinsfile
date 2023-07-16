pipeline {
    agent any 
        stages {
            stage("build started") {
                steps {
                    slackSend channel: 'jenkins', color: 'green', failOnError: true, message: "STARTED: Job ${env.JOB_NAME} [${env.BUILD_NUMBER}]", notifyCommitters: false, teamDomain: 'the-satya'
                }
            }
            stage("code checkout") {
                steps {
                    git branch: 'gameoflife-release-1.0.1', url: 'https://github.com/wakaleo/game-of-life.git'
                }
            }
            stage("Unit Test") {
                steps {
                    sh 'mvn test'
                }
            }
            stage("Integration Test") {
                steps {
                    sh 'mvn verify -DskipUnitTests'
                }
            }
            stage("SCA Sonar") {
                steps {
                    script{
                        withSonarQubeEnv(credentialsId: 'sonardocker') {
                            sh 'mvn clean package sonar:sonar'
                        }
                    }
                    
                }
            }
            stage("Quality Gates") {
                steps {
                    script{
                        waitForQualityGate abortPipeline: false, credentialsId: 'sonardocker'
                    }
                }
            }
        }
    post {
        success {
            slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")

        }
        failure {
            cleanWS()
            slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        } 
    }    
}