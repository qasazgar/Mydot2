
pipeline {
    agent any

    environment {
        NAJVA_SENDER = '1000090990'
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
            echo " CHECK LOGIN TESTS PASSED"
            echo " No SMS will be sent."
            echo "======================================"
        }

        failure {
            echo "======================================"
            echo " CHECK LOGIN TESTS FAILED"
            echo " Sending SMS notification..."
            echo "======================================"

            withCredentials([
                string(
                    credentialsId: '1b704ab0-1c91-45ac-9727-60be462ba1b4',
                    variable: 'NAJVA_TOKEN'
                )
            ]) {

                sh '''
                    set +x

                    cat > send-sms.js <<'EOF'
var request = require('request');

var token = process.env.NAJVA_TOKEN;

var mobiles = [
    process.env.MOBILE_1,
    process.env.MOBILE_2
];

mobiles.forEach(function (mobile) {

    var options = {
        method: 'POST',
        url: 'https://email.najva.com/v1/sms/transactional_sms/',
        headers: {
            'Accept': 'application/json',
            'najva-token': token,
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            sms_content: 'ALERT: MyDot E2E Check Login FAILED. Jenkins Build #' + process.env.BUILD_NUMBER,
            sender: process.env.NAJVA_SENDER,
            mobile: mobile
        })
    };

    request(options, function (error, response) {

        if (error) {
            console.error('SMS Error for ' + mobile + ':', error);
            process.exitCode = 1;
            return;
        }

        console.log('SMS response for ' + mobile + ':');
        console.log(response.body);
    });

});
EOF

                    echo "Sending SMS notification..."

                    node send-sms.js

                    rm -f send-sms.js

                    echo "======================================"
                    echo " SMS notification process completed"
                    echo "======================================"
                '''
            }
        }
    }
}