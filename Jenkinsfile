pipeline {
    agent any

    options {
        disableConcurrentBuilds()
        buildDiscarder(logRotator(
            numToKeepStr: '20',
            artifactNumToKeepStr: '10'
        ))
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
                    echo "===== Environment Check ====="

                    echo "Node version:"
                    node --version

                    echo "NPM version:"
                    npm --version

                    echo "Bruno version:"
                    bru --version
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
                    echo " Running Check login E2E Tests"
                    echo "======================================"

                    bru run "Check login" \
                        --env Dev \
                        --reporter-junit reports/check-login-junit.xml \
                        --reporter-html reports/check-login-report.html
                '''
            }
        }
    }

    post {

        always {
            echo "===== Publishing Test Results ====="

            junit(
                allowEmptyResults: true,
                testResults: 'reports/*-junit.xml'
            )

            archiveArtifacts(
                artifacts: 'reports/*.html',
                allowEmptyArchive: true
            )
        }

        success {
            echo "======================================"
            echo " Check login tests PASSED"
            echo "======================================"
        }

        failure {
            echo "======================================"
            echo " Check login tests FAILED"
            echo " SMS notification should be sent here"
            echo "======================================"

            // SMS notification will be added here
        }
    }
}
