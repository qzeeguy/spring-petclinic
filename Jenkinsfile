pipeline {
  agent {
    docker {
      image 'abhishekf5/maven-abhishek-docker-agent:v1'
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock' // mount Docker socket to access the host's Docker daemon
    }
  }
  stages {
    stage('Checkout') {
      steps {
        sh 'echo passed'
        //git branch: 'main', url: 'https://github.com/qzeeguy/spring-petclinic.gi'
      }
    }
    stage('Build and Test') {
      steps {
        sh 'ls -ltr'
        // build the project and create a JAR file
        sh 'cd spring-petclinic && mvn clean package'
      }
    }
    stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://54.161.90.80:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'qube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'cd java-maven-sonar-argocd-helm-k8s/spring-boot-app && mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }
    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "alqoseemi/ultimate-cicd:${BUILD_NUMBER}"
        // DOCKERFILE_LOCATION = "spring-petclinic/Dockerfile"
        REGISTRY_CREDENTIALS = credentials('dockerhub')
      }
      
     stage('Build & Push Docker Image') {
            steps {
                script {
                    // Build Docker image
                    sh "docker build -t alqoseemi/ultimate-cicd:${BUILD_NUMBER} ."

                    // Push image to Docker Hub
                    def image = docker.image("alqoseemi/ultimate-cicd:${BUILD_NUMBER}")
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

                        # Replace placeholder with current build number
                        sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" spring-app-cicd-manifest/deployment.yml

                        git add spring-app-cicd-manifest/deployment.yml
                        git commit -m "Update image tag to ${BUILD_NUMBER}" || echo "No changes to commit"
                        git push https://${GITHUB_TOKEN}@github.com/phemy0/spring-app-cicd-manifest HEAD:main
                    '''
                }
            }
        }
    }
}
