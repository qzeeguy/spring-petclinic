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
           environment {
               SONAR_URL = "http://54.161.90.80:9000"
           }
           steps {
               withCredentials([string(credentialsId: 'qube', variable: 'SONAR_AUTH_TOKEN')]) {
                   sh 'mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL} -Dcheckstyle.skip=true'
        }
    }
}


        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    def image = docker.image("${DOCKER_IMAGE}")
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
                        image.push()
                    }
                }
            }
        }

        stage('Update Deployment Manifest') {
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config user.email "phemyolowo@gmail.com"
                        git config user.name "phemy0"

                        sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" spring-app-cicd-manifest/deployment.yml

                        git add spring-app-cicd-manifest/deployment.yml
                        git commit -m "Update image tag to ${BUILD_NUMBER}" || echo "No changes to commit"
                        git push https://${GITHUB_TOKEN}@github.com/phemy0/spring-app-cicd-manifest HEAD:main
                    '''
                }
            }
        }

    } // end stages
} // end pipeline
