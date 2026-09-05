```groovy
pipeline {

    agent any

    // ==========================================
    // Run once every hour
    // ==========================================
    triggers {
        cron('H * * * *')
    }

    // ==========================================
    // Pipeline Options
    // ==========================================
    options {

        // Do not allow two builds at the same time
        disableConcurrentBuilds()

        // Keep last 30 builds
        buildDiscarder(
            logRotator(
                numToKeepStr: '30',
                artifactNumToKeepStr: '30'
            )
        )

        // Maximum execution time
        timeout(
            time: 30,
            unit: 'MINUTES'
        )
    }

    // ==========================================
    // Environment
    // ==========================================
    environment {

        // Bruno environment
        BRUNO_ENV = 'Dev'

        // Bruno collection folder
        BRUNO_COLLECTION = '01- End To End'

        // Report directory
        REPORT_DIR = 'results'

        // Report files
        JUNIT_REPORT = 'results/bruno-junit.xml'
        HTML_REPORT  = 'results/bruno-report.html'
        JSON_REPORT  = 'results/bruno-report.json'

        // ======================================
        // SMS configuration
        // ======================================

        SMS_API_URL = 'https://YOUR-SMS-PROVIDER/api/send'

        SMS_API_TOKEN_CREDENTIAL = 'sms-api-token'
        SMS_MOBILE_CREDENTIAL    = 'sms-mobile-number'
    }


    // ==========================================
    // STAGES
    // ==========================================

    stages {


        // ======================================
        // 1. Checkout
        // ======================================

        stage('Checkout') {

            steps {

                echo '=========================================='
                echo 'Checking out repository'
                echo '=========================================='

                checkout scm
            }
        }


        // ======================================
        // 2. Check Node / NPM
        // ======================================

        stage('Check Environment') {

            steps {

                sh '''
                    echo "Node version:"
                    node --version

                    echo ""
                    echo "NPM version:"
                    npm --version
                '''
            }
        }


        // ======================================
        // 3. Install Bruno
        // ======================================

        stage('Install Bruno CLI') {

            steps {

                sh '''
                    echo "Installing Bruno CLI..."

                    npm install -g @usebruno/cli

                    echo ""
                    echo "Bruno version:"
                    bru --version
                '''
            }
        }


        // ======================================
        // 4. Prepare Reports
        // ======================================

        stage('Prepare Reports') {

            steps {

                sh '''
                    echo "Preparing report directory..."

                    rm -rf results
                    mkdir -p results

                    ls -la results
                '''
            }
        }


        // ======================================
        // 5. Run Bruno E2E
        // ======================================

        stage('Run Bruno E2E Tests') {

            steps {

                echo '=========================================='
                echo 'Starting Bruno E2E Tests'
                echo "Environment: ${BRUNO_ENV}"
                echo '=========================================='

                catchError(
                    buildResult: 'FAILURE',
                    stageResult: 'FAILURE'
                ) {

                    sh """
                        cd "${BRUNO_COLLECTION}"

                        echo "Current directory:"
                        pwd

                        echo ""
                        echo "Running Bruno..."

                        bru run \
                            --env "${BRUNO_ENV}" \
                            --reporter-junit "../${JUNIT_REPORT}" \
                            --reporter-html "../${HTML_REPORT}" \
                            --reporter-json "../${JSON_REPORT}"
                    """
                }
            }
        }


        // ======================================
        // 6. Publish JUnit
        // ======================================

        stage('Publish JUnit Report') {

            steps {

                echo 'Publishing JUnit report...'

                junit(
                    testResults: "${JUNIT_REPORT}",
                    allowEmptyResults: true,
                    keepLongStdio: true
                )
            }
        }


        // ======================================
        // 7. Archive Reports
        // ======================================

        stage('Archive Reports') {

            steps {

                echo 'Archiving Bruno reports...'

                archiveArtifacts(
                    artifacts: 'results/**/*',
                    allowEmptyArchive: true,
                    fingerprint: true
                )
            }
        }
    }


    // ==========================================
    // POST
    // ==========================================

    post {


        // ======================================
        // SUCCESS
        // ======================================

        success {

            echo '=========================================='
            echo '✅ BRUNO E2E TESTS PASSED'
            echo '=========================================='

            echo "Environment: ${env.BRUNO_ENV}"
            echo "Build: #${env.BUILD_NUMBER}"
        }


        // ======================================
        // FAILURE
        // ======================================

        failure {

            echo '=========================================='
            echo '🚨 BRUNO E2E TESTS FAILED'
            echo '=========================================='

            script {

                def buildUrl =
                    env.BUILD_URL ?: 'Jenkins URL unavailable'

                def smsMessage =
                    "E2E TEST FAILED | " +
                    "Environment: ${env.BRUNO_ENV} | " +
                    "Job: ${env.JOB_NAME} | " +
                    "Build: #${env.BUILD_NUMBER} | " +
                    "${buildUrl}"

                echo 'SMS notification should be sent here.'

                /*
                 * SMS API
                 *
                 * Uncomment after configuring your
                 * SMS provider.
                 */

                /*
                withCredentials([

                    string(
                        credentialsId: env.SMS_API_TOKEN_CREDENTIAL,
                        variable: 'SMS_API_TOKEN'
                    ),

                    string(
                        credentialsId: env.SMS_MOBILE_CREDENTIAL,
                        variable: 'SMS_MOBILE'
                    )

                ]) {

                    sh """
                        curl --fail \
                             --silent \
                             --show-error \
                             --request POST \
                             '${SMS_API_URL}' \
                             --header 'Authorization: Bearer '\$SMS_API_TOKEN \
                             --header 'Content-Type: application/json' \
                             --data '{
                                "mobile": "'\$SMS_MOBILE'",
                                "message": "${smsMessage}"
                             }'
                    """
                }
                */
            }
        }


        // ======================================
        // ALWAYS
        // ======================================

        always {

            echo '=========================================='
            echo 'Bruno E2E Monitoring Finished'
            echo '=========================================='

            echo "Job: ${env.JOB_NAME}"
            echo "Build: ${env.BUILD_NUMBER}"
            echo "Result: ${currentBuild.currentResult}"
        }
    }
}
```
