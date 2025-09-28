pipeline {
    agent any
    environment {
        DOCKERHUB = credentials('dockerhub-credentials')
        SONAR_TOKEN = credentials('sonar-token')
        KUBECONFIG_FILE = credentials('kubeconfig')
        GIT_URL = 'https://github.com/oumaima-elkachai/espritPFAapp-devsecops.git'
        GIT_BRANCH = 'main'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${GIT_BRANCH}", credentialsId: 'github-token', url: "${GIT_URL}"
            }
        }

        stage('Build, Test & SonarQube') {
            steps {
                script {
                    def services = [
                        [name: 'eureka-server', key: 'tn.esprit.spring:Eureka-Server'],
                        [name: 'formation-service', key: 'Formation-Service'],
                        [name: 'user-service', key: 'User-Service']
                    ]
                    for (service in services) {
                        dir("back/${service.name}") {
                            echo "🏗️ Build + Tests + Sonar pour ${service.name}"
                            withSonarQubeEnv('sonar-server') {
                                sh """
                                    mvn clean verify -Pci sonar:sonar \
                                    -Dsonar.projectKey=${service.key} \
                                    -Dsonar.host.url=${env.SONAR_HOST_URL} \
                                    -Dsonar.login=${SONAR_TOKEN} \
                                    -Dsonar.java.binaries=target/classes \
                                    -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
                                """
                            }
                        }
                    }
                }
            }
        }

        stage('Wait for SonarQube Quality Gate') {
            steps { waitForQualityGate abortPipeline: true }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    def services = ['eureka-server', 'formation-service', 'user-service']
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials') {
                        for (service in services) {
                            def imageName = "oumaimakachai/${service}:latest"
                            sh "docker build -t ${imageName} back/${service}"
                            sh "docker push ${imageName}"
                        }
                    }
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                script {
                    def images = ['oumaimakachai/eureka-server','oumaimakachai/formation-service','oumaimakachai/user-service']
                    for (image in images) {
                        sh "trivy image --exit-code 1 --severity HIGH,CRITICAL ${image}:latest || true"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh 'kubectl apply -f k8s/backend/'
                    sh 'kubectl get pods'
                }
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
            sh 'docker system prune -f || true'
        }
        success { echo '✅ Pipeline réussie !' }
        failure { echo '❌ Pipeline échouée !' }
    }
}
