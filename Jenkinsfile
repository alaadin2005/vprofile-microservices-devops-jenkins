```groovy
pipeline {
    agent {
        kubernetes {
            defaultContainer 'kaniko'
            yaml '''
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: jenkins-kaniko-agent
spec:
  serviceAccountName: jenkins

  containers:

  - name: kaniko
    image: gcr.io/kaniko-project/executor:v1.23.2-debug
    command:
      - /busybox/cat
    tty: true
    volumeMounts:
      - name: docker-config
        mountPath: /kaniko/.docker

  - name: kubectl
    image: bitnami/kubectl:latest
    command:
      - cat
    tty: true

  - name: helm
    image: alpine/helm:3.15.2
    command:
      - cat
    tty: true

  volumes:
    - name: docker-config
      secret:
        secretName: dockerhub-secret
'''
        }
    }

    environment {
        DOCKERHUB_USER = 'alaadin2005'
        APP_IMAGE      = 'vprofileapp'
        DB_IMAGE       = 'vprofiledb'
        TAG            = "${BUILD_NUMBER}"
        NAMESPACE      = 'default'
        RELEASE_NAME   = 'vprofile'
    }

    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '15'))
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
                    dir('Docker-files/app') {
                        sh '''
                        /kaniko/executor \
                          --context=$(pwd) \
                          --dockerfile=Dockerfile \
                          --destination=${DOCKERHUB_USER}/${APP_IMAGE}:${TAG} \
                          --destination=${DOCKERHUB_USER}/${APP_IMAGE}:latest
                        '''
                    }
                }
            }
        }

        stage('Build & Push DB Image') {
            steps {
                container('kaniko') {
                    dir('Docker-files/db') {
                        sh '''
                        /kaniko/executor \
                          --context=$(pwd) \
                          --dockerfile=Dockerfile \
                          --destination=${DOCKERHUB_USER}/${DB_IMAGE}:${TAG} \
                          --destination=${DOCKERHUB_USER}/${DB_IMAGE}:latest
                        '''
                    }
                }
            }
        }

        stage('Validate Kubernetes') {
            steps {
                container('kubectl') {
                    sh '''
                    kubectl version --client
                    kubectl get nodes
                    '''
                }
            }
        }

        stage('Deploy with Helm') {
            steps {
                container('helm') {
                    sh '''
                    helm upgrade --install ${RELEASE_NAME} ./helm/vprofile \
                      --namespace ${NAMESPACE} \
                      --create-namespace \
                      --set app.image.repository=${DOCKERHUB_USER}/${APP_IMAGE} \
                      --set app.image.tag=${TAG} \
                      --set db.image.repository=${DOCKERHUB_USER}/${DB_IMAGE} \
                      --set db.image.tag=${TAG} \
                      --wait \
                      --timeout 5m
                    '''
                }
            }
        }

        stage('Verify Rollout') {
            steps {
                container('kubectl') {
                    sh '''
                    kubectl rollout status deployment/vproapp -n ${NAMESPACE} --timeout=180s
                    kubectl get pods -n ${NAMESPACE}
                    '''
                }
            }
        }
    }

    post {

        success {
            echo "✅ Production deployment completed successfully."
        }

        failure {
            echo "❌ Pipeline failed. Starting rollback..."

            container('helm') {
                sh '''
                helm rollback ${RELEASE_NAME} || true
                '''
            }
        }

        always {
            cleanWs()
        }
    }
}
```
