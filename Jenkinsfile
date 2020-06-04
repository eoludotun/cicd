pipeline {
    environment {
        DEPLOY = "${env.BRANCH_NAME == "master" || env.BRANCH_NAME == "develop" ? "true" : "false"}"
        NAME = "${env.BRANCH_NAME == "master" ? "example" : "example-staging"}"
        VERSION = readMavenPom().getVersion()
        DOMAIN = 'localhost'
        REGISTRY = 'davidcampos/k8s-jenkins-example'
        REGISTRY_CREDENTIAL = 'dockerhub-msatia'
    }
    agent {
        kubernetes {
            defaultContainer 'jnlp'
            yamlFile 'build.yaml'
        }
    }
    stages {
       /* stage('Build') {
            steps {
                container('maven') {
                    sh 'mvn package'
                }
            }
        }*/
         stage('Verify Argo') {
            steps {
                container('argocdcli') {
                    //sh 'argocd version'
                    sh '''
                    ARGOCD_SERVER="localhost"
                    
                    withCredentials([string(credentialsId: "argo-secret-token", variable: 'ARGOCD_AUTH_TOKEN')]) {
                        
                        argocd app sync guestbook --force
                        
                    }'''
                   // sh "echo testing"
                }
            }
        }
        stage('Docker Build') {
            when {
                environment name: 'DEPLOY', value: 'true'
            }
            steps {
                container('docker') {
                    sh "docker build -t ${REGISTRY}:${VERSION} ."
                }
            }
        }
        stage('Docker Publish') {
            when {
                environment name: 'DEPLOY', value: 'true'
            }
            steps {
                container('docker') {
                    withDockerRegistry([credentialsId: "${REGISTRY_CREDENTIAL}", url: ""]) {
                        sh "docker push ${REGISTRY}:${VERSION}"
                    }
                }
            }
        }
        stage('Kubernetes Deploy') {
            when {
                environment name: 'DEPLOY', value: 'true'
            }
            steps {
                container('helm') {
                    sh "helm upgrade --install --force --set name=${NAME} --set image.tag=${VERSION} --set domain=${DOMAIN} ${NAME} ./helm"
                }
            }
        }
    }
}
