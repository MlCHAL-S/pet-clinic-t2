pipeline {
    agent any

    stages {
        stage('Verify') {
            steps {
                sh '''
                    echo "Running verification..."
                    ./mvnw verify
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