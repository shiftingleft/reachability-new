
pipeline {
    agent any

    environment {
        ARTEFACT_NAME = "${WORKSPACE}/target/WebGoat-${BUILD_VERSION}.war"
        BUILD_VERSION = 1
        IQ_SCAN_URL = ""
        IQ_STAGE = "build"

        // Amend these
        IQ_APPNAME = "webgoat-sd2"
        JENKINS_CREDS_ID = "iqserver"
    }

    tools {
        maven 'M3'
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn -B -Dproject.version=$BUILD_VERSION -Dmaven.test.failure.ignore clean package'
            }
            post {
                success {
                    echo 'Now archiving...'
                    archiveArtifacts artifacts: "**/target/*.war"
                }
            }
        }

        stage('Nexus IQ Scan'){
            steps {
                script{
                     nexusPolicyEvaluation( 
                                            failBuildOnNetworkError: true, 
                                            iqApplication: selectedApplication("${IQ_APPNAME}"), 
                                            iqScanPatterns: [[scanPattern: '**/*.war']], 
                                            iqStage: "${IQ_STAGE}", 
                                            jobCredentialsId: "${JENKINS_CREDS_ID}",
                                            callflow: [
                                                enable: true,
                                                failOnError: false,
                                                timeout: '10 minutes',
                                                logLevel: 'DEBUG',
                                                entrypointStrategy: [
                                                    $class: 'NamedStrategy',
                                                    name: 'ACCESSIBLE_CONCRETE'
                                                ]
                            ]
                     )
                }
            }
        }
    }
}

