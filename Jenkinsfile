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
        // stage('Dependency Scan') {
        //     agent { label 'java17jdk' }
        //     steps {
        //         sh '''
        //             echo "Running dependency scan..."
        //             ./mvnw org.owasp:dependency-check-maven:check
        //         '''
        //     }
        // }
        stage('Push to Nexus') {
            agent { label 'java17jdk' }
            steps {
                withCredentials([
                    usernamePassword(credentialsId: 'NEXUS_CREDS', usernameVariable: 'NEXUS_USERNAME', passwordVariable: 'NEXUS_PASSWORD')
                ]) {
                    configFileProvider([configFile(fileId: 'nexus-maven-settings', variable: 'MAVEN_SETTINGS')]) {
                        sh '''
                            echo "Deploying to Nexus..."
                            ./mvnw -s "$MAVEN_SETTINGS" clean deploy -DskipTests
                        '''
                    }
                }
            }
        }
        stage('Pull binary from Nexus') {
            agent { label 'dockercli' }
            steps {
                checkout scm
                withCredentials([
                    usernamePassword(credentialsId: 'NEXUS_CREDS', usernameVariable: 'NEXUS_USERNAME', passwordVariable: 'NEXUS_PASSWORD')
                ]) {
                    configFileProvider([configFile(fileId: 'nexus-maven-settings', variable: 'MAVEN_SETTINGS')]) {
                        sh '''
                            echo "Pulling binary from Nexus..."
                            mkdir -p target
                            ./mvnw -s "$MAVEN_SETTINGS" \
                                org.apache.maven.plugins:maven-dependency-plugin:3.10.0:copy \
                                -Dartifact=org.springframework.samples:spring-petclinic:4.0.0-SNAPSHOT:jar \
                                -DoutputDirectory=target \
                                -Dmdep.stripVersion=true

                            echo "Contents of target directory:"
                            ls -la target
                        '''
                    }
                }
                script {
                    def shortSha = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    sh """
                        echo "Building Docker image..."
                        docker buildx build \
                            --platform linux/amd64 \
                            --load \
                            -t petclinic:${shortSha} .
                    """
                }
            }
        }
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