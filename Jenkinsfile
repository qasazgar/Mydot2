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

        stage('Run Check Login Tests') {
            steps {
                sh '''
                    echo "======================================"
                    echo " Running Check Login E2E Tests"
                    echo "======================================"

                    bru run "Check login" \
                        --env Dev \
                        --reporter-junit reports/check-login-junit.xml \
                        --reporter-html reports/check-login-report.html

                    echo "======================================"
                    echo " Check Login Tests Completed"
                    echo "======================================"
                '''
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
            echo " CHECK LOGIN TESTS PASSED"
            echo " No SMS will be sent."
            echo "======================================"
        }

        failure {
            echo "======================================"
            echo " CHECK LOGIN TESTS FAILED"
            echo " Running SendSmsFail..."
            echo "======================================"

            sh '''
                    echo "======================================"
                    echo " Running Check Login E2E Tests"
                    echo "======================================"

                    bru run "sms" \
            '''

            echo "======================================"
            echo " SendSmsFail completed"
            echo "======================================"
        }
    }
}