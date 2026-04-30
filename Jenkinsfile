pipeline {
    agent {
        kubernetes {
            defaultContainer 'jnlp'
            yaml '''
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins

  containers:

  - name: kaniko
    image: gcr.io/kaniko-project/executor:latest
    command:
      - /busybox/cat
    tty: true
    volumeMounts:
      - name: kaniko-secret
        mountPath: /kaniko/.docker

  - name: kubectl
    image: bitnami/kubectl:latest
    command: ["cat"]
    tty: true

  volumes:
    - name: kaniko-secret
      secret:
        secretName: dockerhub
'''
        }
    }

    environment {
        DOCKERHUB_USER = 'alaadin2005'
        APP_IMAGE = 'vprofileapp'
        DB_IMAGE  = 'vprofiledb'
        TAG = "${BUILD_NUMBER}"
        REGISTRY = "docker.io"
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
                container('kaniko') {
                    sh """
                    /kaniko/executor \
                    --context=`pwd`/Docker-files/app \
                    --dockerfile=`pwd`/Docker-files/app/Dockerfile \
                    --destination=${DOCKERHUB_USER}/${APP_IMAGE}:${TAG} \
                    --cache=true
                    """
                }
            }
        }

        stage('Build & Push DB Image') {
            steps {
                container('kaniko') {
                    sh """
                    /kaniko/executor \
                    --context=`pwd`/Docker-files/db \
                    --dockerfile=`pwd`/Docker-files/db/Dockerfile \
                    --destination=${DOCKERHUB_USER}/${DB_IMAGE}:${TAG} \
                    --cache=true
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
            echo "✅ Kaniko Pipeline Success: Build + Push + Deploy completed"
        }

        failure {
            echo "❌ Pipeline Failed"
        }
    }
}
