pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                sh '''
                    echo "Running tests..."
                    ./mvnw clean test
                '''
            }
        }
    }
}