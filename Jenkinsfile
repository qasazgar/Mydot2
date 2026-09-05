```groovy
pipeline {

    agent any

    // Run automatically once every hour
    triggers {
        cron('H * * * *')
    }

    options {

        // Timestamper plugin
        timestamps()

        // Prevent two executions at the same time
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

    environment {

        // Bruno environment
        BRUNO_ENV = 'Dev'

        // Bruno collection directory
        BRUNO_COLLECTION = '01- End To End'

        // Reports
        REPORT_DIR   = 'results'
        JUNIT_REPORT = 'results/bruno-junit.xml'
        HTML_REPORT  = 'results/bruno-report.html'
        JSON_REPORT  = 'results/bruno-report.json'

        // Jenkins credentials
        // Create these in:
        // Manage Jenkins -> Credentials

        SMS_API_TOKEN_CREDENTIAL = 'sms-api-token'
        SMS_MOBILE_CREDENTIAL    = 'sms-mobile-number'

        // Your SMS API
        SMS_API_URL = 'https://YOUR-SMS-PROVIDER/api/send'
    }


    stages {

        // ==================================================
        // 1. Checkout
        // ==================================================

        stage('Checkout') {
            steps {

                echo 'Checking out source code...'

                checkout scm
            }
        }


        // ==================================================
        // 2. Check Node
        // ==================================================

        stage('Check Environment') {
            steps {

                sh '''
                    echo "Node version:"
                    node --version

                    echo "NPM version:"
                    npm --version
                '''
            }
        }


        // ==================================================
        // 3. Install Bruno CLI
        // ==================================================

        stage('Install Bruno CLI') {
            steps {

                sh '''
                    npm install -g @usebruno/cli

                    echo "Bruno version:"
                    bru --version
                '''
            }
        }


        // ==================================================
        // 4. Prepare Reports
        // ==================================================

        stage('Prepare Reports') {
            steps {

                sh '''
                    rm -rf results
                    mkdir -p results
                '''
            }
        }


        // ==================================================
        // 5. Run Bruno E2E Tests
        // ==================================================

        stage('Run Bruno E2E Tests') {

            steps {

                echo "=========================================="
                echo "Starting Bruno E2E Tests"
                echo "Environment: ${BRUNO_ENV}"
                echo "=========================================="

                /*
                 * catchError allows the pipeline to continue
                 * so reports can still be published even when
                 * Bruno tests fail.
                 */

                catchError(
                    buildResult: 'FAILURE',
                    stageResult: 'FAILURE'
                ) {

                    sh """
                        cd "${BRUNO_COLLECTION}"

                        bru run \
                            --env "${BRUNO_ENV}" \
                            --reporter-junit "../${JUNIT_REPORT}" \
                            --reporter-html "../${HTML_REPORT}" \
                            --reporter-json "../${JSON_REPORT}"
                    """
                }
            }
        }


        // ==================================================
        // 6. Publish JUnit
        // ==================================================

        stage('Publish JUnit Report') {

            steps {

                junit(
                    testResults: "${JUNIT_REPORT}",
                    allowEmptyResults: true,
                    keepLongStdio: true
                )
            }
        }


        // ==================================================
        // 7. Archive Reports
        // ==================================================

        stage('Archive Reports') {

            steps {

                archiveArtifacts(
                    artifacts: 'results/**/*',
                    allowEmptyArchive: true,
                    fingerprint: true
                )
            }
        }
    }


    // ======================================================
    // POST ACTIONS
    // ======================================================

    post {

        // --------------------------------------------------
        // SUCCESS
        // --------------------------------------------------

        success {

            echo '''
            ==========================================
            ✅ BRUNO E2E TESTS PASSED
            ==========================================
            '''

            echo "Environment: ${env.BRUNO_ENV}"
            echo "Build: #${env.BUILD_NUMBER}"

            // No SMS when everything passes
        }


        // --------------------------------------------------
        // FAILURE
        // --------------------------------------------------

        failure {

            echo '''
            ==========================================
            🚨 BRUNO E2E TESTS FAILED
            ==========================================
            '''

            script {

                def buildUrl = env.BUILD_URL ?: 'Jenkins URL unavailable'

                def smsMessage =
                    "🚨 E2E TEST FAILED | " +
                    "Environment: ${env.BRUNO_ENV} | " +
                    "Job: ${env.JOB_NAME} | " +
                    "Build: #${env.BUILD_NUMBER} | " +
                    "${buildUrl}"

                echo "Sending SMS notification..."

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
            }
        }


        // --------------------------------------------------
        // ALWAYS
        // --------------------------------------------------

        always {

            echo '''
            ==========================================
            Bruno E2E Monitoring Finished
            ==========================================
            '''

            echo "Job: ${env.JOB_NAME}"
            echo "Build: #${env.BUILD_NUMBER}"
            echo "Result: ${currentBuild.currentResult}"
        }
    }
}
```
