pipeline {
    agent none

    environment {
        REPO_NAME = credentials('REPO_NAME')
        PROJECT_ID = credentials('PROJECT_ID')
        SERVICE_ACCOUNT_NAME = credentials('GCLOUD_SA')
        REGISTRY_HOST = 'europe-west1-docker.pkg.dev'
        IMAGE_NAME = 'petclinic'
    }

    stages {
        stage('Init metadata') {
            agent { label 'java17jdk' }
            steps {
                script {
                    echo 'Initializing metadata...'
                    env.SHORT_SHA = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                }
                echo "SHORT_SHA=${env.SHORT_SHA}"
            }
        }

        stage('Verify') {
            agent { label 'java17jdk' }
            steps {
                sh '''
                    echo 'Running verification...'
                    ./mvnw clean verify
                '''
            }
        }

        stage('SonarQube') {
            agent { label 'java17jdk' }
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh """
                        echo 'Running SonarQube analysis...'
                        ./mvnw org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
                            -Dsonar.projectKey=petclinic \
                            -Dsonar.projectName='petclinic'
                    """
                }
            }
        }

        stage('Dependency Scan') {
            agent { label 'java17jdk' }
            steps {
                sh '''
                    echo "Running dependency scan..."
                    ./mvnw org.owasp:dependency-check-maven:check
                '''
            }
        }

        stage('Push to Nexus') {
            agent { label 'java17jdk' }
            steps {
                withCredentials([
                    usernamePassword(credentialsId: 'NEXUS_CREDS', usernameVariable: 'NEXUS_USERNAME', passwordVariable: 'NEXUS_PASSWORD')
                ]) {
                    configFileProvider([configFile(fileId: 'nexus-maven-settings', variable: 'MAVEN_SETTINGS')]) {
                        sh '''
                            echo "Deploying to Nexus..."
                            ./mvnw -s "$MAVEN_SETTINGS" deploy -DskipTests
                        '''
                    }
                }

                echo "Stashing artifacts for Docker build..."
                stash name: 'app-jar', includes: 'target/*.jar'
                stash name: 'docker-files', includes: 'Dockerfile,.dockerignore'
            }
        }

        stage('Build image') {
            agent { label 'dockercli' }
            steps {
                unstash 'app-jar'
                unstash 'docker-files'

                script {
                    echo "Building Docker image..."

                    sh """
                        docker buildx build \
                            --platform linux/amd64 \
                            --load \
                            -t ${REGISTRY_HOST}/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:${env.SHORT_SHA} .
                    """
                }
            }
        }

        stage('Push Image') {
            agent { label 'dockercli' }
            steps {
                script {
                    def image = "${REGISTRY_HOST}/${PROJECT_ID}/${REPO_NAME}/${IMAGE_NAME}:${env.SHORT_SHA}"

                    withCredentials([file(credentialsId: 'GCLOUD_CRED', variable: 'GCP_KEY_FILE')]) {
                        sh """
                            echo "Authenticating with GCP..."
                            gcloud auth activate-service-account "${SERVICE_ACCOUNT_NAME}" --key-file="$GCP_KEY_FILE"

                            echo "Configuring Docker to use GCP credentials..."
                            gcloud auth configure-docker ${REGISTRY_HOST} --quiet

                            echo "Pushing Docker image to registry..."
                            docker push ${image}
                        """
                    }
                }
            }
        }

    }
}