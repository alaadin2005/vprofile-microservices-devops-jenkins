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
    env:
      - name: DOCKER_HOST
        value: tcp://localhost:2375

  - name: dind
    image: docker:27-dind
    securityContext:
      privileged: true
    env:
      - name: DOCKER_TLS_CERTDIR
        value: ""
    args:
      - --host=tcp://0.0.0.0:2375
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
        DOCKERHUB_USER = 'alaadin2005'
        APP_IMAGE = 'vprofileapp'
        DB_IMAGE  = 'vprofiledb'
        TAG = "${BUILD_NUMBER}"
        DOCKER_HOST = 'tcp://localhost:2375'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/abdelrahmanonline4/dockerized-microservices.git'
            }
        }

        stage('Docker Login') {
            steps {
                container('docker') {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh '''
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        '''
                    }
                }
            }
        }

        stage('Build App Image') {
            steps {
                container('docker') {
                    dir('Docker-files/app') {
                        sh """
                            docker build -t ${DOCKERHUB_USER}/${APP_IMAGE}:${TAG} .
                        """
                    }
                }
            }
        }

        stage('Build DB Image') {
            steps {
                container('docker') {
                    dir('Docker-files/db') {
                        sh """
                            docker build -t ${DOCKERHUB_USER}/${DB_IMAGE}:${TAG} .
                        """
                    }
                }
            }
        }

        stage('Push Images') {
            steps {
                container('docker') {
                    sh """
                        docker push ${DOCKERHUB_USER}/${APP_IMAGE}:${TAG}
                        docker push ${DOCKERHUB_USER}/${DB_IMAGE}:${TAG}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                container('kubectl') {
                    sh '''
                        kubectl apply -f app-secret.yml
                        kubectl apply -f db-CIP.yml
                        kubectl apply -f mc-CIP.yml
                        kubectl apply -f mcdep.yml
                        kubectl apply -f rmq-CIP-service.yml
                        kubectl apply -f rmq-dep.yml
                        kubectl apply -f vproapp-service.yml
                        kubectl apply -f vproappdep.yml
                        kubectl apply -f vprodbdep.yml
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline Success: Build + Push + Deploy completed"
        }

        failure {
            echo "❌ Pipeline Failed"
        }

        always {
            container('docker') {
                sh 'docker logout || true'
            }
        }
    }
}
