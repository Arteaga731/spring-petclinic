pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'Arteaga731/spring-petclinic'
        DOCKER_TAG = 'latest'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']], // Cambiar a '*/master' si es necesario
                    extensions: [],
                    userRemoteConfigs: [[
                        url: 'https://github.com/Arteaga731/spring-petclinic.git',
                        credentialsId: 'github-token', // Verificar que esta credencial existe
                        timeout: 10 // Añadir timeout para evitar esperas infinitas
                    ]]
                ])
            }
        }

        stage('Maven Build') {
            agent {
                docker {
                    image 'maven:3.9.6-eclipse-temurin-17'
                    args '-v $HOME/.m2:/root/.m2'
                    reuseNode true // Reutiliza el mismo workspace
                }
            }
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Docker Build') {
            agent any
            steps {
                script {
                    if (!fileExists('Dockerfile')) {
                        error("Dockerfile no encontrado")
                    }
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                }
            }
        }

        stage('Docker Push') {
            agent any
            when {
                branch 'main' // Cambiar a 'master' si es necesario
            }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline completado - Limpieza de workspace"
            cleanWs() // Limpia el workspace después de la ejecución
        }
    }
}