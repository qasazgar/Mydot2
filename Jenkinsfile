```groovy
pipeline {

    agent any

    // ==========================================
    // Run every hour
    // ==========================================
    triggers {
        cron('H * * * *')
    }

    options {

        // Prevent two executions at the same time
        disableConcurrentBuilds()

        // Keep last 30 builds
        buildDiscarder(
            logRotator(
                numToKeepStr: '30',
                artifactNumToKeepStr: '30'
            )
        )

        // Maximum test execution time
        timeout(
            time: 30,
            unit: 'MINUTES'
        )
    }


    stages {

        // ==========================================
        // 1. Checkout
        // ==========================================

        stage('Checkout') {
            steps {

                checkout scm

            }
        }


        // ==========================================
        // 2. Check Environment
        // ==========================================

        stage('Check Environment') {
            steps {

                sh '''
                    echo "=========================================="
                    echo "Environment Check"
                    echo "=========================================="

                    echo "Node version:"
                    node --version

                    echo ""
                    echo "NPM version:"
                    npm --version

                    echo ""
                    echo "Bruno version:"
                    bru --version

                    echo ""
                    echo "Current directory:"
                    pwd

                    echo ""
                    echo "Repository files:"
                    ls -la
                '''
            }
        }


        // ==========================================
        // 3. Prepare Reports
        // ==========================================

        stage('Prepare Reports') {
            steps {

                sh '''
                    rm -rf reports
                    mkdir -p reports
                '''
            }
        }


        // ==========================================
        // 4. Run Check Login
        // ==========================================

        stage('Run Check Login') {

            steps {

                echo "=========================================="
                echo "Running Check Login"
                echo "Environment: Dev"
                echo "=========================================="

                sh '''
                    bru run "Check login" \
                        --env Dev \
                        --reporter-junit reports/check-login-junit.xml \
                        --reporter-html reports/check-login-report.html
                '''
            }
        }
    }


    // ==========================================
    // POST ACTIONS
    // ==========================================

    post {

        // ==========================================
        // Always
        // ==========================================

        always {

            echo "=========================================="
            echo "Publishing Test Reports"
            echo "=========================================="

            junit(
                allowEmptyResults: true,
                testResults: 'reports/*-junit.xml'
            )

            archiveArtifacts(
                artifacts: 'reports/*.html',
                allowEmptyArchive: true,
                fingerprint: true
            )
        }


        // ==========================================
        // SUCCESS
        // ==========================================

        success {

            echo "=========================================="
            echo "✅ CHECK LOGIN PASSED"
            echo "=========================================="

            echo "Environment: Dev"
            echo "Build: #${env.BUILD_NUMBER}"
        }


        // ==========================================
        // FAILURE
        // ==========================================

        failure {

            echo "=========================================="
            echo "🚨 CHECK LOGIN FAILED"
            echo "=========================================="

            echo "Environment: Dev"
            echo "Build: #${env.BUILD_NUMBER}"
            echo "Job: ${env.JOB_NAME}"

            /*
             * SMS will be added here.
             */

            echo "SMS notification will be sent here."
        }
    }
}
```
