pipeline {

    agent any

    options {
        skipDefaultCheckout(true)
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "===== Building Docker Image ====="

                    docker build \
                        -t greenleaf-plant-store:${BUILD_NUMBER} .

                    echo "===== Image Created ====="

                    docker images greenleaf-plant-store
                '''
            }
        }

        stage('Test Docker Image') {
            steps {
                sh '''
                    echo "===== Starting Test Container ====="

                    docker run -d \
                        --name greenleaf-test-${BUILD_NUMBER} \
                        -p 80 \
                        greenleaf-plant-store:${BUILD_NUMBER}

                    echo "===== Waiting for Nginx ====="
                    sleep 5

                    echo "===== Container Status ====="
                    docker ps -a | grep greenleaf-test-${BUILD_NUMBER} || true

                    echo "===== Container Logs ====="
                    docker logs greenleaf-test-${BUILD_NUMBER} || true

                    echo "===== Container Port ====="
                    docker port greenleaf-test-${BUILD_NUMBER} || true

                    echo "===== Detecting Assigned Port ====="

                    PORT=$(docker port greenleaf-test-${BUILD_NUMBER} 80/tcp | sed 's/.*://')

                    echo "GreenLeaf is running on host port: ${PORT}"

                    echo "===== Testing Website ====="

                    curl -f http://localhost:${PORT}

                    echo "===== Website Test Successful ====="
                '''
            }
        }

        stage('Cleanup Test Container') {
            steps {
                sh '''
                    echo "===== Cleaning Test Container ====="

                    docker stop greenleaf-test-${BUILD_NUMBER} 2>/dev/null || true

                    docker rm greenleaf-test-${BUILD_NUMBER} 2>/dev/null || true
                '''
            }
        }

        stage('Cleanup Old GreenLeaf Images') {
            steps {
                sh '''
                    echo "===== Cleaning Old GreenLeaf Images ====="

                    docker images \
                        'greenleaf-plant-store' \
                        --format '{{.Tag}}' |
                    grep -v "^${BUILD_NUMBER}$" |
                    xargs -r -I {} docker rmi \
                        greenleaf-plant-store:{} || true

                    echo "===== Remaining GreenLeaf Images ====="

                    docker images greenleaf-plant-store
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
