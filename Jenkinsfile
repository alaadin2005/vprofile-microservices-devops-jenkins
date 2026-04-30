pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: default
  containers:
  - name: jnlp
    image: jenkins/inbound-agent:latest
    tty: true

  - name: kaniko
    image: gcr.io/kaniko-project/executor:latest
    command:
    - /bin/sh
    args:
    - -c
    - cat
    tty: true
    tty: true
    volumeMounts:
    - name: docker-config
      mountPath: /kaniko/.docker

  volumes:
  - name: docker-config
    secret:
      secretName: dockerhub-secret
'''
        }
    }

    environment {
        DOCKERHUB_USER = 'alaadin2005'
        APP_IMAGE = 'vprofileapp'
        DB_IMAGE  = 'vprofiledb'
        TAG = 'latest'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/abdelrahmanonline4/dockerized-microservices.git'
            }
        }

        stage('Build & Push App Image') {
            steps {
                dir('Docker-files/app') {
                    container('kaniko') {
                        sh '''
                        /kaniko/executor \
                          --context `pwd` \
                          --dockerfile Dockerfile \
                          --destination ${DOCKERHUB_USER}/${APP_IMAGE}:${TAG}
                        '''
                    }
                }
            }
        }

        stage('Build & Push DB Image') {
            steps {
                dir('Docker-files/db') {
                    container('kaniko') {
                        sh '''
                        /kaniko/executor \
                          --context `pwd` \
                          --dockerfile Dockerfile \
                          --destination ${DOCKERHUB_USER}/${DB_IMAGE}:${TAG}
                        '''
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
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

    post {
        success {
            echo "✅ Build and deployment completed successfully."
        }
        failure {
            echo "❌ Pipeline failed."
        }
    }
}
