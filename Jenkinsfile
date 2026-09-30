pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                        -t greenleaf-plant-store:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Test Docker Image') {
            steps {
                sh '''
                    docker run -d \
                        --name greenleaf-test-${BUILD_NUMBER} \
                        -p 8081:80 \
                        greenleaf-plant-store:${BUILD_NUMBER}

                    sleep 5

                    curl -f http://localhost:8081

                    docker stop greenleaf-test-${BUILD_NUMBER}
                    docker rm greenleaf-test-${BUILD_NUMBER}
                '''
            }
        }

    }

    post {

        success {
            echo '🌱 GreenLeaf website pipeline completed successfully!'
        }

        failure {
            echo '❌ GreenLeaf pipeline failed.'
        }

        always {
            sh '''
                docker rm -f greenleaf-test-${BUILD_NUMBER} 2>/dev/null || true
            '''
        }
    }
}
