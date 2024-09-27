pipeline {
    agent any
    tools {
        jdk 'java17'
        nodejs 'node18'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        TMDB_V3_API_KEY = credentials('TMDB_V3_API_KEY')
    }
    stages {
        stage('Clean Up Previous WorkSpace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Github') {
            steps {
                git branch: 'master', url: 'https://github.com/YasinDeger48/DevSecOps-Netflix.git'
            }
        }
        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix'''
                }
            }
        }
        stage("SonarQube Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('Install necessary dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage('clean up old docker images') {
            steps {
                sh 'docker image prune -a -f'
            }
        }
        stage("Docker Build & Push"){
            steps{
                withCredentials([usernamePassword(credentialsId: 'DockerHUB-YasinDeger48', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh "docker build --build-arg TMDB_V3_API_KEY=${TMDB_V3_API_KEY} -t yasindeger48/$JOB_NAME:v$BUILD_ID ."
                    sh "docker build --build-arg TMDB_V3_API_KEY=${TMDB_V3_API_KEY} -t yasindeger48/$JOB_NAME:latest ."
                    sh 'docker push yasindeger48/$JOB_NAME:v$BUILD_ID'
                    sh 'docker push yasindeger48/$JOB_NAME:latest'
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image yasindeger48/$JOB_NAME:latest > trivyimage.txt"
            }
        }
        stage('kubernetes apply deployment') {
            steps {
                kubernetesDeploy(
                    configs: 'Kubernetes/deployment.yml',
                    kubeconfigId: 'kubeconfig'
                )
            }
        }
        stage('kubernetes apply service') {
            steps {
                kubernetesDeploy(
                    configs: 'Kubernetes/service.yml',
                    kubeconfigId: 'kubeconfig'
                )
            }
        }
    }
}
