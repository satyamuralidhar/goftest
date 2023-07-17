pipeline {
    agent any 
    parameters {
        string(name: 'username',description: 'dockerHub userName',defaultValue: 'muralidhar123')
        string(name: 'imagename',description: 'Java Application',defaultValue: 'javaapp')
        string(name: 'imagetag',description: 'dockerHub userName',defaultValue: 'latest')

    }
        stages {
            stage("Workspace Clean") {
                steps {
                    script{
                        cleanWS
                    }
                }
            }
            stage("Build Started") {
                steps {
                    slackSend channel: 'jenkins',
                    color: 'green', 
                    failOnError: true, 
                    message: "STARTED: Job ${env.JOB_NAME} [${env.BUILD_NUMBER}]", 
                    notifyCommitters: false, teamDomain: 'the-satya'
                }
            }
            stage("Unit Test") {
                steps {
                    script{
                        sh 'mvn test'
                    }
                }
            }
            stage("Integration Test") {
                steps {
                    script{
                        sh 'mvn verify -DskipUnitTests'
                    }
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
                        waitForQualityGate abortPipeline: false,
                        credentialsId: 'sonardocker'
                    }
                }
            }
            stage("Docker Build") {
                steps {
                    script{
                        dockerBuild("${params.username}","${params.imagename}","${params.imagetag}")
                    }
                }
            }
            stage("Docker Vulnerablity Scan") {
                steps {
                    script{
                        snykScan("${params.username}","${params.imagename}","${params.imagetag}")
                    }
                }
            }
        }
    post {
        success {
            slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")

        }
        failure {
            slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        } 
    }    
}