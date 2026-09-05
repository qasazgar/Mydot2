pipeline {

    agent any

    /*
     * Run once every hour.
     * Jenkins chooses a stable minute automatically.
     *
     * Example:
     * 10:17
     * 11:17
     * 12:17
     * ...
     */
    triggers {
        cron('H * * * *')
    }

    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(
            logRotator(
                numToKeepStr: '30',
                artifactNumToKeepStr: '30'
            )
        )
        timeout(time: 30, unit: 'MINUTES')
    }

    environment {

        /*
         * Bruno environment name
         *
         * Example:
         * environments/Dev.bru
         */
        BRUNO_ENV = 'Dev'

        /*
         * Folder containing Bruno tests.
         *
         * If your repository root itself is the Bruno collection,
         * change this to:
         *
         * BRUNO_COLLECTION = '.'
         */
        BRUNO_COLLECTION = '01- End To End'

        /*
         * Reports
         */
        REPORT_DIR   = 'results'
        JUNIT_REPORT = 'results/bruno-junit.xml'
        HTML_REPORT  = 'results/bruno-report.html'
        JSON_REPORT  = 'results/bruno-report.json'

        /*
         * SMS API
         *
         * Change this to your SMS provider API URL.
         *
         * Examples:
         * Kavenegar
         * Ghasedak
         * Melipayamak
         * Twilio
         * Custom internal SMS service
         */
        SMS_API_URL = 'https://YOUR-SMS-PROVIDER/api/send'

        /*
         * Jenkins credential IDs
         *
         * Create these credentials in:
         *
         * Manage Jenkins
         *   -> Credentials
         *
         * sms-api-token:
         *   Secret text
         *
         * sms-mobile-number:
         *   Secret text
         */
        SMS_API_TOKEN_CREDENTIAL = 'sms-api-token'
        SMS_MOBILE_CREDENTIAL    = 'sms-mobile-number'
    }


    stages {

        /*
         * --------------------------------------------------
         * 1. Checkout
         * --------------------------------------------------
         */
        stage('Checkout') {
            steps {

                echo 'Checking out repository...'

                checkout scm
            }
        }


        /*
         * --------------------------------------------------
         * 2. Check Node / NPM
         * --------------------------------------------------
         */
        stage('Check Environment') {
            steps {

                echo 'Checking Node.js...'

                sh '''
                    node --version
                    npm --version
                '''
            }
        }


        /*
         * --------------------------------------------------
         * 3. Install Bruno CLI
         * --------------------------------------------------
         */
        stage('Install Bruno CLI') {
            steps {

                echo 'Installing Bruno CLI...'

                sh '''
                    npm install -g @usebruno/cli

                    echo "Bruno version:"
                    bru --version
                '''
            }
        }


        /*
         * --------------------------------------------------
         * 4. Prepare Report Folder
         * --------------------------------------------------
         */
        stage('Prepare Reports') {
            steps {

                echo 'Preparing report directory...'

                sh '''
                    rm -rf "${REPORT_DIR}"
                    mkdir -p "${REPORT_DIR}"
                '''
            }
        }


        /*
         * --------------------------------------------------
         * 5. Run Bruno E2E Tests
         * --------------------------------------------------
         */
        stage('Run Bruno E2E Tests') {

            steps {

                echo "Running Bruno E2E tests..."

                /*
                 * catchError is intentional.
                 *
                 * If Bruno exits with code != 0:
                 *
                 * - Pipeline becomes FAILURE
                 * - Remaining report/archive stages can still execute
                 * - post { failure } will send SMS
                 */
                catchError(
                    buildResult: 'FAILURE',
                    stageResult: 'FAILURE'
                ) {

                    sh """
                        cd "${BRUNO_COLLECTION}"

                        echo "=========================================="
                        echo " Bruno E2E Test"
                        echo " Environment : ${BRUNO_ENV}"
                        echo " Build       : ${BUILD_NUMBER}"
                        echo "=========================================="

                        bru run \
                            --env "${BRUNO_ENV}" \
                            --reporter-junit "../${JUNIT_REPORT}" \
                            --reporter-html "../${HTML_REPORT}" \
                            --reporter-json "../${JSON_REPORT}"
                    """
                }
            }
        }


        /*
         * --------------------------------------------------
         * 6. Publish JUnit
         * --------------------------------------------------
         */
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


        /*
         * --------------------------------------------------
         * 7. Archive Reports
         * --------------------------------------------------
         */
        stage('Archive Reports') {

            steps {

                echo 'Archiving Bruno reports...'

                archiveArtifacts(
                    artifacts: "${REPORT_DIR}/**/*",
                    allowEmptyArchive: true,
                    fingerprint: true
                )
            }
        }
    }


    /*
     * ======================================================
     * POST ACTIONS
     * ======================================================
     */
    post {


        /*
         * --------------------------------------------------
         * Always
         * --------------------------------------------------
         */
        always {

            echo '''
            ==========================================
            Bruno E2E Monitoring Finished
            ==========================================
            '''

            echo "Job: ${env.JOB_NAME}"
            echo "Build: ${env.BUILD_NUMBER}"
            echo "Result: ${currentBuild.currentResult}"
        }


        /*
         * --------------------------------------------------
         * Success
         * --------------------------------------------------
         */
        success {

            echo '''
            ==========================================
            ✅ BRUNO E2E TESTS PASSED
            ==========================================
            '''

            /*
             * No SMS on success
             */
        }


        /*
         * --------------------------------------------------
         * Failure
         * --------------------------------------------------
         */
        failure {

            echo '''
            ==========================================
            🚨 BRUNO E2E TEST FAILURE
            Sending SMS...
            ==========================================
            '''

            script {

                /*
                 * Build Jenkins URL
                 */
                def buildUrl = env.BUILD_URL ?: 'Jenkins URL unavailable'

                /*
                 * SMS message
                 */
                def smsMessage =
                    "E2E TEST FAILED | " +
                    "Environment: ${env.BRUNO_ENV} | " +
                    "Job: ${env.JOB_NAME} | " +
                    "Build: #${env.BUILD_NUMBER} | " +
                    "${buildUrl}"


                /*
                 * Read secrets from Jenkins Credentials
                 */
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

                    /*
                     * Generic SMS request.
                     *
                     * Change BODY depending on your SMS provider.
                     */
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


        /*
         * --------------------------------------------------
         * Unstable
         * --------------------------------------------------
         *
         * Jenkins JUnit may mark a build UNSTABLE instead
         * of FAILURE when tests fail.
         *
         * Therefore SMS should also be sent here.
         */
        unstable {

            echo '''
            ==========================================
            ⚠️ BRUNO E2E TESTS UNSTABLE
            Sending SMS...
            ==========================================
            '''

            script {

                def buildUrl = env.BUILD_URL ?: 'Jenkins URL unavailable'

                def smsMessage =
                    "E2E TEST UNSTABLE | " +
                    "Environment: ${env.BRUNO_ENV} | " +
                    "Job: ${env.JOB_NAME} | " +
                    "Build: #${env.BUILD_NUMBER} | " +
                    "${buildUrl}"


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
    }
}