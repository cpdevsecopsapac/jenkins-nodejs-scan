pipeline {
    agent any

    environment {
        // Use Jenkins credentials binding
        CHKP_CLOUDGUARD_ID = credentials('chkp-cloudguard-id')
        CHKP_CLOUDGUARD_SECRET = credentials('chkp-cloudguard-secret')
        SHIFTLEFT_REGION = "eu1"
        SPECTRAL_DSN = credentials('spectral-dsn')
    }

    stages {
        stage('Clone Github repository') {
            steps {
                checkout scm
            }
        }

        stage('install Spectral') {
            steps {
                sh "curl -L 'https://spectral-eu.checkpoint.com/latest/x/sh?dsn=$SPECTRAL_DSN' | sh"
            }
        }

        stage('Pre-Build Spectral Scan for Secrets,Misconfiguration and IaC') {
            steps {
                sh "$HOME/.spectral/spectral scan --engines secrets,oss --include-tags base,audit --ok"
            }
        }

        stage('Docker image Build and scan prep') {
            steps {
                sh 'docker build -t nilsujma/node .'
                sh 'docker save nilsujma/node -o node.tar'
            }
        }

        stage('ShiftLeft Container Image Scan Online') {
            steps {
                script {
                    try {
                        sh 'chmod +x shiftleft'
                        sh './shiftleft image-scan -t 180 -i node.tar -r -2002 -e 4b89d765-1dfd-4c19-bf26-a34374142d42'
                    } catch (Exception e) {
                        echo "ShiftLeft docker image scan failed."
                    }
                }
            }
        }

        stage('Shiftleft Container Image Scan Offline') {
            steps {
                script {
                    try {
                        sh 'chmod +x shiftleft'
                        sh './shiftleft image-scan -t 180 -i node.tar'
                    } catch (Exception e) {
                        echo "ShiftLeft docker Image scan failed."
                    }
                }
            }
        }
        stage('Post-Build Spectral Scan for Secrets,Misconfiguration and IaC') {
            steps {
                sh "$HOME/.spectral/spectral scan --engines secrets,oss --include-tags base,audit --ok"
            }
        }
    }
}
