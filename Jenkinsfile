pipeline {
    agent { label 'podman && c1' }

    triggers {
        pollSCM('H/2 * * * *')
    }

    environment {
        REGISTRY = 'registry.assign.samgain3.300.ops:5000'
        IMAGE_NAME = 'bind-secondary'
    }

    stages {
        stage('Prepare') {
            steps {
                sh 'podman --version'
                sh 'git --version'
            }
        }

        stage('Build') {
            steps {
                withCredentials([file(credentialsId: 'bind-xfr-key', variable: 'XFR_KEY')]) {
                    sh '''
                        cp "$XFR_KEY" ./xfr.key
                        chmod 600 ./xfr.key
                        podman build -t "$REGISTRY/$IMAGE_NAME:$BUILD_NUMBER" .
                        rm -f ./xfr.key
                    '''
                }
            }
        }

        stage('Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'registry-creds', usernameVariable: 'REG_USER', passwordVariable: 'REG_PASS')]) {
         sh '''
                        printf "%s" "$REG_PASS" | podman login "$REGISTRY" --username "$REG_USER" --password-stdin
                        podman push "$REGISTRY/$IMAGE_NAME:$BUILD_NUMBER"
                        podman logout "$REGISTRY"
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'rm -f ./xfr.key || true'
        }
    }
}