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

        stage('CVE Scan (Trivy)') {
            steps {
                sh '''
                    set -eu

                    rm -f trivy-image.tar trivy-report.txt

                    echo "Exporting image for Trivy scan..."
                    podman save --format docker-archive \
                        -o trivy-image.tar \
                        "$REGISTRY/$IMAGE_NAME:$BUILD_NUMBER"

                    podman volume create trivy-cache >/dev/null 2>&1 || true

                    echo "Scanning HIGH and CRITICAL vulnerabilities..."
                    podman run --rm \
                        -v "$PWD:/work" \
                        -v trivy-cache:/root/.cache/trivy \
                        docker.io/aquasec/trivy:0.75.0 \
                        image \
                        --input /work/trivy-image.tar \
                        --scanners vuln \
                        --severity HIGH,CRITICAL \
                        --format table \
                        --output /work/trivy-report.txt \
                        --exit-code 0

                    echo "===== TRIVY HIGH/CRITICAL REPORT ====="
                    cat trivy-report.txt

                    echo "===== TRIVY SUMMARY ====="
                    grep '^Total:' trivy-report.txt || true

                    rm -f trivy-image.tar
                '''

                archiveArtifacts artifacts: 'trivy-report.txt',
                                 fingerprint: true
            }
        }

        stage('Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'registry-creds',
                    usernameVariable: 'REG_USER',
                    passwordVariable: 'REG_PASS'
                )]) {
                    sh '''
                        printf "%s" "$REG_PASS" | podman login "$REGISTRY" \
                            --username "$REG_USER" --password-stdin

                        podman push "$REGISTRY/$IMAGE_NAME:$BUILD_NUMBER"

                        podman logout "$REGISTRY"
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'rm -f ./xfr.key ./trivy-image.tar || true'
        }
    }
}
