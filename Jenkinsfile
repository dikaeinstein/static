pipeline {
    agent any
    stages {
        stage('Lint HTML') {
            steps {
                sh 'tidy -q -e *.html'
            }
        }
        stage('Upload to AWS') {
            steps {
                withAWS(region:'eu-west-2',credentials:'aws-static') {
                sh 'echo "Uploading content with AWS creds"'
                    s3Upload(pathStyleAccessEnabled: true, payloadSigningEnabled: true, file:'index.html', bucket:'dika-static-jenkins-pipeline')
                }
            }
        }
        stage('Test Deployment') {
            steps {
                sh 'echo "Testing website deployment"'
                sh '''
                    url='http://dika-static-jenkins-pipeline.s3-website.eu-west-2.amazonaws.com'
                    timeout=2
                    online=false

                    for (( i=1; i<=3; i++ ))
                    do
                      code=`curl -sL --connect-timeout 20 --max-time 30 -w "%{http_code}\\n" "$url" -o /dev/null`

                      echo "Found code $code for $url."

                      if [ "$code" = "200" ]; then
                        echo "Website $url is online."
                        online=true
                        break
                      else
                        echo "Website $url seems to be offline. Waiting $timeout seconds."
                        sleep $timeout
                      fi
                    done

                    if $online; then
                      echo "Deployment succeeded"
                      exit 0
                    else
                      echo "Deployment failed."
                      exit 1
                    fi
                '''
            }
        }
    }
}
