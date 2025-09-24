pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('sonar-token')
        GITHUB_TOKEN = credentials('githubb-token')
        PATH = "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
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
                    def services = ['Eureka-Server', 'User-Service', 'Formation-Service']
                    for (service in services) {
                        dir("back/${service}") {
                            echo "Building ${service}..."
                            sh 'mvn clean install -DskipTests'
                        }
                    }
                }
            }
        }

        stage('Start MySQL') {
            steps {
                dir('back') {
                    echo 'Starting MySQL...'
                    sh 'docker compose up -d'
                    sh 'sleep 20' // attendre que MySQL soit prêt
                }
            }
        }

        stage('Test Backend Microservices') {
            steps {
                script {
                    def services = ['Eureka-Server', 'User-Service', 'Formation-Service']
                    for (service in services) {
                        dir("back/${service}") {
                            echo "Testing ${service}..."
                            sh 'mvn test'
                        }
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def services = ['Eureka-Server', 'User-Service', 'Formation-Service']
                    for (service in services) {
                        dir("back/${service}") {
                            withSonarQubeEnv('SonarQube') {
                                echo "Analyzing ${service} in SonarQube..."
                                sh "mvn sonar:sonar -Dsonar.login=$SONAR_TOKEN"
                            }
                        }
                    }
                }
            }
        }

        stage('Docker Build Backend') {
            steps {
                script {
                    def services = ['Eureka-Server', 'User-Service', 'Formation-Service']
                    for (service in services) {
                        dir("back/${service}") {
                            echo "Building Docker image for ${service}..."
                            sh "docker build -t espritapp-${service.toLowerCase()}:latest ."
                        }
                    }
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                script {
                    def images = ['espritapp-eureka-server', 'espritapp-user-service', 'espritapp-formation-service']
                    for (image in images) {
                        echo "Scanning Docker image ${image}..."
                        sh "trivy image ${image}:latest"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                dir('k8s/backend') {
                    sh 'kubectl apply -f .'
                }
            }
        }

        stage('Stop MySQL') {
            steps {
                dir('back') {
                    echo 'Stopping MySQL...'
                    sh 'docker compose down'
                }
            }
        }
    }
}
