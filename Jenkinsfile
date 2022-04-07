pipeline {
    agent any
    environment {
        JENKINS_NODE_COOKIE="dontKillMe"
    }
    stages {
      stage('Build') {
            steps {
                git 'https://github.com/ericsheridanwhs/owasp-juiceshop'
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
                  sh "pkill node"
              }
          }
      }
   }
}