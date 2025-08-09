# Projeto DevOps: Pipeline de CI/CD com Jenkins e Docker

## Introdução
Este projeto foi desenvolvimento como proposta de trabalho da disciplina de Cisco DevNet Associate da Pós-graduação de Engenharia de Serviços e Sistemas de Cloud Computing (PUCPR), focada em DevOps.

Teve como objetivo praticar alguns conceitos e práticas estudados teoriamente na matéria, através do desenvolvimento de uma pipeline de CI/CD utilizando Jenkins e Docker, contendo os seguintes estágios:
1. Cleanup
2. Checkout
3. Build
4. Delivery

### Estrutura do Projeto

```
├── Jenkinsfile
├── db
    ├── Dockerfile
    └── codigo.sql
└── web
    ├── Dockerfile
    ├── main.py
    └── requirements.txt
```

<img a="https://img.shields.io/badge/Jenkins-49728B?style=for-the-badge&logo=jenkins&logoColor=white">
<img a="https://img.shields.io/badge/Docker-2CA5E0?style=for-the-badge&logo=docker&logoColor=white">

## Preparação

### Subida do Jenkins localmente

Para realizar as atividades do trabalho, foi necessário, como preparação das ferramentas necessárias, a subida do Jenkins no ambiente local, de modo a poder criar e executar a pipeline desenvolvida.

Para isso, foi utilizado o comando abaixo no terminal do Linux WSL, para permitir o *build* e demais operações do Docker dentro do container do Jenkins:

```
docker run -d --name jenkins -u root -p 8080:8080 -p 50000:50000 -v jenkins-data:/var/jenkins_home -v $(which docker):/usr/bin/docker -v /var/run/docker.sock:/var/run/docker.sock jenkins/jenkins:lts-jdk17
```

Para validar a subida e pegar as credenciais do usuário administrador inicial, os comandos foram acompanhados com: ```docker logs -f jenkins```

Após esse container subir com sucesso, o Jenkins pôde ser acesso através do navegador no endereço https://localhost:8080

### Repositório e Token de Acesso do Docker Hub

A próxima ferramenta que será envolvida na atividade é o [Docker Hub](hub.docker.com/), para *push* das imagens que serão construídas nos últimos estágios da pipeline. 

Foram necessários, para tanto, a criação de dois repositórios para cada uma das imagens (db e web), além de um Token de acesso, o qual foi adicionado como credencial de acesso no Jenkins, com permissões de "Read, Write, Delete".

## Desenvolvimento

### Estrutura da Pipeline

#### 1. Cleanup

O primeiro estágio, chamado "Cleanup", corresponde à limpeza do Woskspace (diretório de trabalho do Jenkins), utilizando o plugin [Workspace Cleanup](https://plugins.jenkins.io/ws-cleanup):

```
stage('Cleanup') {
    steps {
        echo "Realizando a limpeza do Workspace..."
        cleanWs()
    }
}
```

#### 2. Checkout

A seguir, o "Checkout" realiza um clone de um repositório Git indicado, com o conteúdo de uma determinada *branch*, através do plugin [Git](https://www.jenkins.io/doc/pipeline/steps/git):

```
stage('Checkout') {
    steps {
        echo "Realizando o Checkout do repositório do Git..."
        git branch: 'master', url: 'https://github.com/brambillagabrielle/pucpr-devops-project'
    }
}
```

#### 3. Build

Após isso, com o conteúdo do repositório no diretório de trabalho do Jenkins, o passo "Build" realiza a construção de ambas as imagens com o Docker.

Conforme indicado pela estrutura do projeto, são indicados cada um dos 2 diretórios, que possuem Dockerfiles indicando o passo a passo para empacotamento da aplicação e do DB, e passando um "nome:tag" baseado no repositório do Docker Hub e o número do build do Jenkins:

```
stage('Build') {
    steps {
        echo "Realizando o Build das imagens do Docker..."
        sh "docker build -t brambillagabi/imagem-db:${BUILD_NUMBER} ./db"
        sh "docker build -t brambillagabi/imagem-web:${BUILD_NUMBER} ./web"
    }
}
```

#### 4. Delivery

Por fim, após as imagens estarem prontas, o último estágio, de "Delivery", é utilizado para envio das imagens aos repositórios do Docker Hub, realizando o login com as credenciais criadas e registradas para esse fim:

```
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
```