pipeline {
    agent any

    stages {
        stage('Verify') {
            steps {
                sh '''
                    echo "Running verification..."
                    ./mvnw clean verify
                '''
            }
        }
        stage('SonarQube') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh """
                        ./mvnw org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
                            -Dsonar.projectKey=petclinic \
                            -Dsonar.projectName='petclinic'
                    """
                }
            }
        }
        stage('Dependency Scan') {
            steps {
                sh '''
                    echo "Running dependency scan..."
                    ./mvnw org.owasp:dependency-check-maven:check
                '''
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def shortSha = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    sh """
                        echo "Building Docker image..."
                        docker build -t petclinic:${shortSha} .
                    """
                }
            }
        }
    }
}