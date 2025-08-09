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
                echo "Realizando o build das imagens do Docker..."
                sh 'docker build -t brambillagabi/imagem-db:latest ./db'
                sh 'docker build -t brambillagabi/imagem-web:latest ./web'
            }
        }
        stage('Delivery') {
            steps {
                echo "Realizando o Delivery das imagens do Docker..."
                sh 'docker push brambillagabi/imagem-db:latest'
                sh 'docker push brambillagabi/imagem-web:latest'
            }
        }
    }
}
