pipeline {
    agent any

    stages {
        stage('Validate') {
            steps {
                sh '''
                    echo "Validating..."
                    ./mvnw validate
                '''
            }
        }
        stage('Test') {
            steps {
                sh '''
                    echo "Running tests..."
                    ./mvnw clean test
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
    }
}