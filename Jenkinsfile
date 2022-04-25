pipeline {
    agent any
    triggers { pollSCM('* * * * *') }
    environment {
        JENKINS_NODE_COOKIE="dontKillMe"
    }
    stages {
        stage('Setup') {
            steps {
                sh 'sudo npm install -g newman'
            }
        }
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }
        stage('IntegrationTests') {
            steps {
                // start target app
                sh "npm start &"
                sh "while ! curl -s -o /dev/null http://127.0.0.1:3000/rest/user/login; do sleep 1; done"

                // run newman integration tests against target app using postman collections
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
                // (optional) only required if vantage prevent is not already installed on host
                // sh "wget https://github.com/whitehatsec/vantage-prevent-distributions/releases/latest/download/VantagePrevent.deb"
                // sh "sudo apt install -y -f ./VantagePrevent.deb"

                // start target app
                sh "npm start &"
                sh "while ! curl -s -o /dev/null http://127.0.0.1:3000/rest/user/login; do sleep 1; done"

                // run vantage prevent against target app using postman collections as input
                sh """
                    for f in test/postman/*.postman_collection.json; do
                        [ -f "\$f" ] || continue
                        echo running security test: "\$f"
                        dast-attacker-cli -allowed-host http://localhost:3000 -license-key=${WHITEHAT_LICENSE_KEY} -fail-on-severity=high "\$f"
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
