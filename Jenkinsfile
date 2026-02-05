pipeline {
    agent any

    environment {
        IMAGE_NAME = "jenkins-jfrog-aws"
        IMAGE_TAG  = "1.0"
        ARTIFACTORY_URL = "trial9wr9k6.jfrog.io/docker-trial"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Validate Files') {
            steps {
                sh 'test -f index.html'
                sh 'test -f Dockerfile'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh 'docker tag $IMAGE_NAME:$IMAGE_TAG $ARTIFACTORY_URL/$IMAGE_NAME:$IMAGE_TAG'
            }
        }

        stage('Login to JFrog') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'jfrog-cred',
                        usernameVariable: 'JFROG_USER',
                        passwordVariable: 'JFROG_PASS'
                    )
                ]) {
                    sh 'docker login $ARTIFACTORY_URL -u $JFROG_USER -p $JFROG_PASS'
                }
            }
        }

        stage('Push Image to JFrog') {
            steps {
                sh 'docker push $ARTIFACTORY_URL/$IMAGE_NAME:$IMAGE_TAG'
            }
        }

        stage('Run Container') {
            steps {
                sh 'docker rm -f jenkins_jfrog || true'
                sh 'docker run -d -p 8090:80 --name jenkins_jfrog $ARTIFACTORY_URL/$IMAGE_NAME:$IMAGE_TAG'
            }
        }

        stage('Smoke Test') {
            steps {
                sh 'curl http://localhost:8090'
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully"
        }
        failure {
            echo "Pipeline failed"
        }
    }
}
