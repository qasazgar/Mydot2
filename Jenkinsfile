pipeline {
    agent any

    options {
        disableConcurrentBuilds()

        buildDiscarder(
            logRotator(
                numToKeepStr: '20',
                artifactNumToKeepStr: '10'
            )
        )
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check Environment') {
            steps {
                sh '''
                    echo "======================================"
                    echo " Environment Check"
                    echo "======================================"

                    echo "Node version:"
                    node --version

                    echo "NPM version:"
                    npm --version

                    echo "Bruno version:"
                    bru --version

                    echo "======================================"
                '''
            }
        }

        stage('Prepare Reports') {
            steps {
                sh '''
                    rm -rf reports
                    mkdir -p reports
                '''
            }
        }

        stage('Run CheckWallet Tests') {
            steps {
                catchError(
                    buildResult: 'FAILURE',
                    stageResult: 'FAILURE'
                ) {
                    sh '''
                        echo "======================================"
                        echo " Running CheckWallet E2E Tests"
                        echo "======================================"

                        bru run "CheckWallet" \
                            --env Dev \
                            --reporter-junit reports/check-wallet-junit.xml \
                            --reporter-html reports/check-wallet-report.html

                        echo "======================================"
                        echo " CheckWallet Tests Completed"
                        echo "======================================"
                    '''
                }
            }
        }

        stage('Run CheckLogin Tests') {
            steps {
                catchError(
                    buildResult: 'FAILURE',
                    stageResult: 'FAILURE'
                ) {
                    sh '''
                        echo "======================================"
                        echo " Running CheckLogin E2E Tests"
                        echo "======================================"

                        bru run "CheckLogin" \
                            --env Dev \
                            --reporter-junit reports/check-login-junit.xml \
                            --reporter-html reports/check-login-report.html

                        echo "======================================"
                        echo " CheckLogin Tests Completed"
                        echo "======================================"
                    '''
                }
            }
        }
    }

    post {

        always {
            echo "======================================"
            echo " Publishing Test Results"
            echo "======================================"

            junit(
                allowEmptyResults: true,
                testResults: 'reports/*-junit.xml'
            )

            archiveArtifacts(
                artifacts: 'reports/*.html',
                allowEmptyArchive: true
            )

            echo "======================================"
        }

        success {
            echo "======================================"
            echo " CHECKWALLET + CHECKLOGIN TESTS PASSED"
            echo " No SMS will be sent."
            echo "======================================"
        }

        failure {
            echo "======================================"
            echo " CHECKWALLET OR CHECKLOGIN TESTS FAILED"
            echo " Running SendSmsFail..."
            echo "======================================"

            sh '''
                echo "======================================"
                echo " Running SMS Failure Notification"
                echo "======================================"

                bru run "sms" \
                    --env Dev

                echo "======================================"
                echo " SendSmsFail completed"
                echo "======================================"
            '''
        }
    }
}