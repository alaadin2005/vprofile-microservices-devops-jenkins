pipeline {
    agent {
        kubernetes {
            label 'vprofile-agent'

            yaml """
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins
  containers:
  - name: docker
    image: docker:27-cli
    command:
    - cat
    tty: true
    volumeMounts:
    - name: docker-sock
      mountPath: /var/run/docker.sock
    - name: workspace-volume
      mountPath: /home/jenkins/agent

  - name: kubectl
    image: bitnami/kubectl:latest
    command:
    - cat
    tty: true
    volumeMounts:
    - name: workspace-volume
      mountPath: /home/jenkins/agent

  volumes:
  - name: docker-sock
    hostPath:
      path: /var/run/docker.sock

  - name: workspace-volume
    emptyDir: {}
"""
        }
    }

    environment {
        IMAGE_NAME = "alaadin2005/vprofileapp"
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/alaadin2005/vprofile-microservices-devops-jenkins.git'
            }
        }

        stage('Docker Build') {
            steps {
                container('docker') {
                    sh '''
                    docker version
                    docker build -t $IMAGE_NAME:$IMAGE_TAG .
                    '''
                }
            }
        }

        stage('Docker Login') {
            steps {
                container('docker') {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'USER',
                        passwordVariable: 'PASS'
                    )]) {
                        sh '''
                        echo $PASS | docker login -u $USER --password-stdin
                        '''
                    }
                }
            }
        }

        stage('Push Image') {
            steps {
                container('docker') {
                    sh '''
                    docker push $IMAGE_NAME:$IMAGE_TAG
                    docker tag $IMAGE_NAME:$IMAGE_TAG $IMAGE_NAME:latest
                    docker push $IMAGE_NAME:latest
                    '''
                }
            }
        }

        stage('Deploy to K8s') {
            steps {
                container('kubectl') {
                    sh '''
                    kubectl set image deployment/vprofile \
                    vprofile=$IMAGE_NAME:$IMAGE_TAG \
                    -n default

                    kubectl rollout status deployment/vprofile -n default
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline Success'
        }

        failure {
            echo '❌ Pipeline Failed'
        }

        always {
            container('docker') {
                sh 'docker logout || true'
            }
        }
    }
}
