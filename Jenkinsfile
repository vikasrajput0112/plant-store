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

            echo "===== Container Status ====="
            docker ps -a | grep greenleaf-test-${BUILD_NUMBER} || true

            echo "===== Container Logs ====="
            docker logs greenleaf-test-${BUILD_NUMBER} || true

            echo "===== Docker Port ====="
            docker port greenleaf-test-${BUILD_NUMBER} || true

            echo "===== Curl Test ====="
            curl -v http://localhost:8081 || true

            docker stop greenleaf-test-${BUILD_NUMBER} || true
            docker rm greenleaf-test-${BUILD_NUMBER} || true
        '''
    }
}
        stage('Cleanup Old GreenLeaf Images') {
    steps {
        sh '''
            docker images \
                'greenleaf-plant-store' \
                --format '{{.Tag}}' |
            grep -v "^${BUILD_NUMBER}$" |
            xargs -r -I {} docker rmi greenleaf-plant-store:{} || true
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
