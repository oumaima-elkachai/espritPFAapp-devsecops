pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('sonar-token')
        GITHUB_TOKEN = credentials('githubb-token')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'githubb-token',
                    url: 'https://github.com/oumaima-elkachai/espritPFAapp-devsecops.git'
            }
        }

        stage('Build Backend Microservices') {
            steps {
                script {
                    def services = ['Eureka-Server', 'Formation-Service', 'User-Service']
                    for (service in services) {
                        dir("back/${service}") {
                            echo "Building ${service}..."
                            sh 'mvn clean install -DskipTests'

                        }
                    }
                }
            }
        }

        stage('Test Backend Microservices') {
            steps {
                script {
                    def services = ['Eureka-Server', 'Formation-Service', 'User-Service']
                    for (service in services) {
                        dir("back/${service}") {
                            echo "Testing ${service}..."
                            sh './mvnw test'
                        }
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def services = ['Eureka-Server', 'Formation-Service', 'User-Service']
                    for (service in services) {
                        dir("back/${service}") {
                            withSonarQubeEnv('SonarQube') {
                                echo "Analyzing ${service} in SonarQube..."
                                sh "./mvnw sonar:sonar -Dsonar.login=$SONAR_TOKEN"
                            }
                        }
                    }
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir('front') {
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }

        stage('Docker Build Backend') {
            steps {
                script {
                    def services = ['Eureka-Server', 'Formation-Service', 'User-Service']
                    for (service in services) {
                        dir("back/${service}") {
                            echo "Building Docker image for ${service}..."
                            sh "docker build -t espritapp-${service.toLowerCase()}:latest ."
                        }
                    }
                }
            }
        }

        stage('Docker Build Frontend') {
            steps {
                dir('front') {
                    echo "Building Docker image for frontend..."
                    sh "docker build -t espritapp-frontend:latest ."
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                script {
                    def images = ['espritapp-eureka-server', 'espritapp-formation-service', 'espritapp-user-service', 'espritapp-frontend']
                    for (image in images) {
                        echo "Scanning Docker image ${image}..."
                        sh "trivy image ${image}:latest"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s/backend/'
                sh 'kubectl apply -f k8s/frontend/'
            }
        }
    }
}
