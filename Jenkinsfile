pipeline {

    agent any

    tools {
        jdk 'Java17'
        maven 'M398'
    }

    environment {
        DOCKER_IMAGE = "prajnashetty529/register-app"
        GIT_COMMIT = "${env.GIT_COMMIT}"
    }

    stages {

        stage("Build Application") {
            steps {
                sh "mvn clean package"
            }
        }

        stage("Test Application") {
            steps {
                sh "mvn test"
            }
        }

        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonar-qube-token') {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-qube-token'
                }
            }
        }

        stage("Build Docker Image") {
            steps {
                sh 'printenv'
                sh "docker build -t ${DOCKER_IMAGE}:${GIT_COMMIT} ."
                sh "docker tag ${DOCKER_IMAGE}:${GIT_COMMIT} ${DOCKER_IMAGE}:latest"
            }
        }

        stage("Push Docker Image") {
            steps {
                withCredentials([string(credentialsId: 'DOCKER_PAT', variable: 'DOCKER_PAT')]) {
                    sh '''
                        echo "Logging in to Docker Hub..."
                        echo "$DOCKER_PAT" | docker login -u prajnashetty529 --password-stdin

                        echo "Pushing image with commit tag..."
                        docker push ${DOCKER_IMAGE}:${GIT_COMMIT}

                        echo "Pushing image with latest tag..."
                        docker push ${DOCKER_IMAGE}:latest
                    '''
                }
            }
        }

        stage("Trivy Scan") {
            steps {
                sh '''
                    trivy image --exit-code 1 --severity HIGH,CRITICAL \
                        --format json --output trivy-report.json \
                        --no-progress ${DOCKER_IMAGE}:latest || echo "Vulnerabilities found"
                '''
            }
        }
    }
}
