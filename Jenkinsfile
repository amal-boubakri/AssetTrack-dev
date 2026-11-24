pipeline {
    agent any

    options {
        timestamps()
    }

    stages {
        stage('1. Checkout') {
            steps {
                checkout scm
            }
        }

        stage('2. Backend (CI)') {
            steps {
                dir('backend') {
                    bat '''
                        npm ci
                        npm test --if-present
                    '''
                }
            }
        }

        stage('3. Frontend (CI)') {
            steps {
                dir('frontend') {
                    bat '''
                        npm ci
                        npm run build
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'AssetTrack CI : succes.'
        }
        failure {
            echo 'AssetTrack CI : echec. Consulter la console Jenkins.'
        }
    }
}