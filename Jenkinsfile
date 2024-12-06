pipeline {
    agent any

    environment {
        ARTEFACT_NAME = "${WORKSPACE}/target/WebGoat-${BUILD_VERSION}.war"
        BUILD_VERSION = 1
        IQ_SCAN_URL = ""
        IQ_STAGE = "build"
        IQ_APPNAME = ".reachability-v2"
        JENKINS_CREDS_ID = "iq" 
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

        stage('Nexus IQ Scan') {
            steps {
                script {
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
                            logLevel: 'INFO',
                            algorithm: 'RTA_PLUS',  // Specify the algorithm as RTA_PLUS
                            entrypointStrategy: [
                                $class: 'NamedStrategy',
                                name: 'JAVA_MAIN',
                                namespaces: [
                                    ''
                                ]
                            ]
                        ]
                    )
                }
            }
        }
    }
}
