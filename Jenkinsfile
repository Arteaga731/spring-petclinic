pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'Arteaga731/spring-petclinic'
        DOCKER_TAG = 'latest'
    }

    stages {
        // Eliminé el doble checkout que aparecía en el log
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    extensions: [],
                    userRemoteConfigs: [[
                        url: 'https://github.com/Arteaga731/spring-petclinic.git',
                        credentialsId: 'github-token' // Asegúrate de crear esta credencial
                    ]]
                ])
            }
        }

        stage('Maven Build') {
            agent {
                docker {
                    image 'maven:3.9.6-eclipse-temurin-17'
                    args '-v $HOME/.m2:/root/.m2'
                    reuseNode true
                    // Eliminé withDockerRegistry ya que no es necesario para pull de imágenes públicas
                }
            }
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Docker Build') {
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
            when {
                branch 'main'
            }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-creds', // Asegúrate de crear esta credencial
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
            deleteDir() // Usamos deleteDir en lugar de cleanWs
        }
        failure {
            echo "Pipeline fallido - Enviar notificación"
            // Configura aquí tus notificaciones (Slack, Email, etc.)
        }
    }
}