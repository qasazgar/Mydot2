pipeline {
    agent any

    environment {
        NAJVA_SENDER = '90008239'
        MOBILE_1     = '09127988405'
        MOBILE_2     = '09127988406'
    }

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
            echo " No SMS will be sent."
            echo "======================================"
        }

       failure {
    echo "======================================"
    echo " Check login tests FAILED"
    echo " Sending SMS notification..."
    echo "======================================"

    withCredentials([
        string(
            credentialsId: 'najva-sms-token',
            variable: 'NAJVA_TOKEN'
        )
    ]) {

        sh '''
            set +x

            MESSAGE="ALERT: MyDot E2E Check Login FAILED. Jenkins Build #${BUILD_NUMBER}. Please check Jenkins."

            curl --fail-with-body -sS \
                -X POST \
                "https://email.najva.com/v1/sms/transactional_sms/" \
                -H "Accept: application/json" \
                -H "najva-token: ${NAJVA_TOKEN}" \
                -H "Content-Type: application/json" \
                --data "{
                    \\"sms_content\\": \\"${MESSAGE}\\",
                    \\"sender\\": \\"${NAJVA_SENDER}\\",
                    \\"mobile\\": \\"${MOBILE_1}\\"
                }"

            curl --fail-with-body -sS \
                -X POST \
                "https://email.najva.com/v1/sms/transactional_sms/" \  
                -H "Accept: application/json" \
                -H "najva-token: ${NAJVA_TOKEN}" \
                -H "Content-Type: application/json" \
                --data "{
                    \\"sms_content\\": \\"${MESSAGE}\\",
                    \\"sender\\": \\"${NAJVA_SENDER}\\",
                    \\"mobile\\": \\"${MOBILE_2}\\"
                }"
        '''
    }
}
    }
}