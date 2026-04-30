```groovy id="7fefg8"
pipeline {
    agent {
        kubernetes {
            defaultContainer 'docker'
            yaml '''
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
        mountPath: /var/run

  - name: dind
    image: docker:27-dind
    securityContext:
      privileged: true
    env:
      - name: DOCKER_TLS_CERTDIR
        value: ""
    volumeMounts:
      - name: docker-sock
        mountPath: /var/run

  - name: kubectl
    image: bitnami/kubectl:latest
    command:
      - cat
    tty: true

  volumes:
    - name: docker-sock
      emptyDir: {}
'''
        }
    }

    environment {
        DOCKER_HOST = 'tcp://localhost:2375'
        DOCKERHUB_USER = 'alaadin2005'
        APP_IMAGE = 'vprofileapp'
        DB_IMAGE = 'vprofiledb'
        TAG = 'latest'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/abdelrahmanonline4/dockerized-microservices.git'
            }
        }

        stage('Build App') {
            steps {
                dir('Docker-files/app') {
                    sh 'docker build -t ${DOCKERHUB_USER}/${APP_IMAGE}:${TAG} .'
                }
            }
        }

        stage('Build DB') {
            steps {
                dir('Docker-files/db') {
                    sh 'docker build -t ${DOCKERHUB_USER}/${DB_IMAGE}:${TAG} .'
                }
            }
        }

        stage('Push Images') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'alaadin2005',
                    passwordVariable: 'Alaadin@2013'
                )]) {
                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    docker push ${DOCKERHUB_USER}/${APP_IMAGE}:${TAG}
                    docker push ${DOCKERHUB_USER}/${DB_IMAGE}:${TAG}
                    docker logout
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                container('kubectl') {
                    sh 'kubectl get pods'
                }
            }
        }
    }
}
```
