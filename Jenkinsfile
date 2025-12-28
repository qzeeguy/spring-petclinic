pipeline {
    agent {
        docker {
            image 'maven:3.9.9-eclipse-temurin-17'
            args '--user root \
                  -v /var/run/docker.sock:/var/run/docker.sock \
                  -v /opt/jenkins/ssl:/ssl'
        }
    }

    environment {
        SONAR_URL = "https://54.161.90.80"
    }

    stages {

        stage('Checkout Source') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/qzeeguy/spring-petclinic.git'
            }
        }

        stage('Build (Skip Checkstyle)') {
            steps {
                sh '''
                    mvn clean package \
                    -DskipTests=true \
                    -Dcheckstyle.skip=true
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'qube', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh '''
                        # Define the correct certificate file path
                        CERT_FILE="/ssl/sonar.crt"

                        # Import the certificate only if it exists
                        if [ -f "$CERT_FILE" ]; then
                            if ! keytool -list -cacerts -storepass changeit -alias sonar >/dev/null 2>&1; then
                                echo "Importing SonarQube certificate..."
                                keytool -import -trustcacerts -alias sonar \
                                    -file "$CERT_FILE" \
                                    -cacerts -storepass changeit -noprompt
                            fi
                        else
                            echo "WARNING: Certificate file not found at $CERT_FILE, skipping import..."
                        fi

                        # Run SonarQube analysis
                        mvn sonar:sonar \
                            -Dcheckstyle.skip=true \
                            -Dsonar.login=$SONAR_AUTH_TOKEN \
                            -Dsonar.host.url=$SONAR_URL
                            -Dsonar.ws.ssl.verify=false
                    '''
                }
            }
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
