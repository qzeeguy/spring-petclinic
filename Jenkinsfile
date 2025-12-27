pipeline {
    agent {
        docker {
            image 'maven:3.9.9-eclipse-temurin-17'
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    stages {

        stage('Clean') {
            steps {
                sh 'rm -rf spring-petclinic'
            }
        }

        stage('Clone') {
            steps {
                git branch: 'main', url: 'https://github.com/qzeeguy/spring-petclinic.git'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn clean install -DskipTests=true'
            }
        }

 stage('Static Code Analysis') {
    environment {
        SONAR_URL = "http://127.0.0.1:9000"
    }
    steps {
        withCredentials([string(credentialsId: 'sonar', variable: 'SONAR_AUTH_TOKEN')]) {
            // Secure: shell interprets the token
            sh 'mvn clean verify sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=$SONAR_URL'
        }
    }
}


      stage('Build and Push Docker Image') {
            steps {
                script {
                    sh '''
                        cd spring-petclinic
                        docker build -t alqoseemi/ultimate-cicd:${BUILD_NUMBER} .
                    '''
                    def dockerImage = docker.image("alqoseemi/ultimate-cicd:${BUILD_NUMBER}")
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Update Deployment File') {
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config user.email "phemyolowo@gmail.com"
                        git config user.name "phemy0"

                        sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" spring-app-cicd-manifest/deployment.yml

                        git add spring-app-cicd-manifest/deployment.yml
                        git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                        git push https://${GITHUB_TOKEN}@github.com/phemy0/spring-app-cicd-manifest HEAD:main
                    '''
                }
            }
        }
    }
}
