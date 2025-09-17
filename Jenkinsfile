pipeline {
    agent any

    tools {
        maven 'Maven3'
        jdk 'Java17'
    }

    environment {
        SONAR_TOKEN = credentials('sonar-token')
        GITHUB_TOKEN = credentials('githubb-token') // updated ID
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'githubb-token', // updated ID
                    url: 'https://github.com/oumaima-elkachai/espritPFAapp-devsecops.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "mvn sonar:sonar -Dsonar.login=$SONAR_TOKEN"
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t espritapp:latest .'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh 'trivy image espritapp:latest'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s/'
            }
        }
    }
}
