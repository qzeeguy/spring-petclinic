pipeline {
    agent {
        docker {
            image 'maven:3.9.9-eclipse-temurin-17'
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        SONAR_URL = "http://54.161.90.80:9000"
        DOCKER_IMAGE = "alqoseemi/ultimate-cicd:${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                sh 'echo passed'
                // git branch: 'main', url: 'https://github.com/qzeeguy/spring-petclinic.git'
            }
        }

        stage('Prepare Maven Cache') {
            steps {
                sh '''
                    echo "Cleaning corrupted Maven cache..."
                    rm -rf ~/.m2/repository/org/testcontainers
                    rm -rf ~/.m2/repository/tools/jackson
                    rm -rf ~/.m2/repository/.cache
                '''
            }
        }

        stage('Build and Test') {
            steps {
                sh '''
                    mvn clean verify \
                        -Dcheckstyle.skip=true \
                        -Dspring.profiles.active=test \
                        -Dspring.docker.compose.enabled=false \
                        -DskipTests=false \
                        -Dtest='!*MySqlIntegrationTests,!*PostgresIntegrationTests'
                '''
            }
        }

        stage('Static Code Analysis') {
            steps {
                withCredentials([string(credentialsId: 'qube', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh """
                        mvn sonar:sonar \
                            -Dsonar.token=$SONAR_AUTH_TOKEN \
                            -Dsonar.host.url=${SONAR_URL} \
                            -Dcheckstyle.skip=true \
                            -Dsonar.sources=src/main/java \
                            -Dsonar.plugins.downloadOnlyRequired=true
                    """
                }
            }
        }

        /* ======== ONLY CHANGE STARTS HERE ======== */

         stage('Build Docker Image') {
             steps {  
                   sh '''
                       echo "Installing Docker CLI..."
                       apt-get update -qq
                       apt-get install -y -qq docker.io

                       docker build -t ${DOCKER_IMAGE} .
                      '''
          }
     }

        /* ======== ONLY CHANGE ENDS HERE ======== */

        stage('Push Docker Image') {
            steps {
                script {
                    def image = docker.image("${DOCKER_IMAGE}")
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-creds') {
                        image.push()
                    }
                }
            }
        }

        stage('Update Deployment Manifest') {
            steps {
                withCredentials([string(credentialsId: 'github-manifest-token', variable: 'GITHUB_TOKEN')]) {
                    sh '''
       git config --global user.email "phemyolowo@gmail.com"
       git config --global user.name "phemy0"

       # Remove old clone if exists
       rm -rf spring-app-cicd-manifest

       # Clone the repo using token
       git clone https://x-access-token:${GITHUB_TOKEN}@github.com/phemy0/spring-app-cicd-manifest.git
       cd spring-app-cicd-manifest

       sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" deployment.yml

       git add deployment.yml
       git commit -m "Update image tag to ${BUILD_NUMBER}" || echo "No changes to commit"
       git push origin main
                   '''
        }
    }
}

    }
}
