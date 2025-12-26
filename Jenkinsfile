
pipeline {
  agent {
    docker {
      image 'maven:3.9.9-eclipse-temurin-17'
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock' // mount Docker socket to access the host's Docker daemon
    }
       }

    }
    stage('Code') {
        stage('clean') {
            sh """ #!/bin/bash
                rm -rf spring-petclinic
            """
        }
        stage('clone') {
            git branch: 'main', url: 'https://github.com/qzeeguy/spring-petclinic.git'
        } // stage: clone
        stage('compile') {
            sh """ #!/bin/bash
                mvn clean install -DskipTests=true
            """
        } // stage: compile
     stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://127.0.0.1:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'sonar', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'cd spring-petclinic && mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }
    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "alqoseemi/ultimate-cicd:${BUILD_NUMBER}"
        // DOCKERFILE_LOCATION = "java-maven-sonar-argocd-helm-k8s/spring-boot-app/Dockerfile"
        REGISTRY_CREDENTIALS = credentials('dockerhub')
      }
      steps {
        script {
            sh 'cd spring-petclinic && docker build -t ${DOCKER_IMAGE} .'
            def dockerImage = docker.image("${DOCKER_IMAGE}")
            docker.withRegistry('https://index.docker.io/v1/', "dockerhub") {
                dockerImage.push()
            }
        }
      }
    }
    stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "spring-app-cicd-manifest"
            GIT_USER_NAME = "phemy0"
        }
        steps {
            withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "phemyolowo@gmail.com"
                    git config user.name "phemy0"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" spring-app-cicd-manifest/deployment.yml
                    git add spring-app-cicd-manifests/deployment.yml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
            }
        }
    }
  }
