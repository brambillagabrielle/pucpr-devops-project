pipeline {
    agent any
    stages {
        stage('Cleanup') {
            steps {
                echo "Realizando a limpeza do Workspace..."
                cleanWs()
            }
        }
        stage('Checkout') {
            steps {
                echo "Realizando o Checkout do repositório do Git..."
                git branch: 'master', url: 'https://github.com/brambillagabrielle/pucpr-devops-project'
            }
        }
        stage('Build') {
            steps {
                echo "Realizando o Build das imagens do Docker..."
                sh "docker build -t brambillagabi/imagem-db:${BUILD_NUMBER} ./db"
                sh "docker build -t brambillagabi/imagem-web:${BUILD_NUMBER} ./web"
            }
        }
        stage('Delivery') {
            steps {
                echo "Realizando o Delivery das imagens do Docker..."
                withCredentials([usernamePassword(credentialsId: 'docker-token',
                                   usernameVariable: 'DOCKER_USER',
                                   passwordVariable: 'DOCKER_PASS')]) {
                  sh '''echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'''
                  sh "docker push brambillagabi/imagem-db:${BUILD_NUMBER}"
                  sh "docker push brambillagabi/imagem-web:${BUILD_NUMBER}"
                }
            }
        }
    }
}