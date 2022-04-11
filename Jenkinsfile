pipeline {
    agent any
    triggers { pollSCM('* * * * *') }
    environment {
        JENKINS_NODE_COOKIE="dontKillMe"
    }
    stages {
        stage('Build') {
            steps {
                sh 'npm install --force'
            }
        }
        stage('IntegrationTests') {
            steps {
                sh "npm start &"
                sh "while ! curl -s -o /dev/null http://127.0.0.1:3000/rest/user/login; do sleep 1; done"
                sh """
                    for f in test/postman/*.postman_collection.json; do
                        [ -f "\$f" ] || continue
                        echo running integration test: "\$f"
                        newman run "\$f"
                    done
                """
            }
            post {
                always {
                    sh "pkill node || true"
                }
            }
        }
        stage('SecurityTests') {
            steps {
                sh "npm start &"
                sh "while ! curl -s -o /dev/null http://127.0.0.1:3000/rest/user/login; do sleep 1; done"
                sh """
                    for f in test/postman/*.postman_collection.json; do
                        [ -f "\$f" ] || continue
                        echo running security test: "\$f"
                        dast-attacker-cli -allowed-host http://localhost:3000 -license-key=$(WHITEHAT_LICENSE_KEY) -fail-on-severity=high "\$f"
                    done
                """
            }
            post {
                always {
                    sh "pkill node || true"
                }
            }
        }
    }
}
