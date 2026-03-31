pipeline {
    agent none

    environment {
        REPO_NAME = credentials('REPO_NAME')
        PROJECT_ID = credentials('PROJECT_ID')
    }

    stages {
        // stage('Verify') {
        //     agent { label 'java17jdk' }
        //     steps {
        //         sh '''
        //             echo "Running verification..."
        //             ./mvnw clean verify
        //         '''
        //     }
        // }
        // stage('SonarQube') {
        //     agent { label 'java17jdk' }
        //     steps {
        //         withSonarQubeEnv('sonarqube') {
        //             sh """
        //                 ./mvnw org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
        //                     -Dsonar.projectKey=petclinic \
        //                     -Dsonar.projectName='petclinic'
        //             """
        //         }
        //     }
        // }
        stage('Dependency Scan') {
            agent { label 'java17jdk' }
            steps {
                sh '''
                    echo "Running dependency scan..."
                    ./mvnw org.owasp:dependency-check-maven:check
                '''
            }
        }
        // stage('Push to Nexus') {
        //     agent { label 'java17jdk' }
        //     steps {
        //         withCredentials([
        //             usernamePassword(credentialsId: 'NEXUS_CREDS', usernameVariable: 'NEXUS_USERNAME', passwordVariable: 'NEXUS_PASSWORD')
        //         ]) {
        //             configFileProvider([configFile(fileId: 'nexus-maven-settings', variable: 'MAVEN_SETTINGS')]) {
        //                 sh '''
        //                     echo "Deploying to Nexus..."
        //                     ./mvnw -s $MAVEN_SETTINGS clean deploy -DskipTests
        //                 '''
        //             }
        //         }
                
        //     }
        // }
        // stage('Build Docker Image') {
        //     agent { label 'dockercli' }
        //     steps {
        //         script {
        //             def shortSha = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        //             sh """
        //                 echo "Building Docker image..."
        //                 docker build -t petclinic:${shortSha} .
        //             """
        //         }
        //     }
        // }
        // stage('Push Image') {
        //     agent { label 'dockercli' }
        //     steps {
        //         script {
        //             def shortSha = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        //             def image = "europe-west1-docker.pkg.dev/${PROJECT_ID}/${REPO_NAME}/petclinic:${shortSha}"

        //             withCredentials
        //             sh """
        //                 echo "Pushing Docker image..."
        //                 docker push ${image}
        //             """
        //         }
        //     }
        // }
    }
}