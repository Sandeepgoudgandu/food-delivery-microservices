pipeline {
    agent any

    environment {
        REGISTRY = "your-docker-registry"       // example: docker.io/sandeep123
        SONAR_HOST = "http://your-sonar-url:9000"
        SONAR_TOKEN = credentials('sonar-token')
        NEXUS_URL = "http://your-nexus:8081/repository/maven-releases/"
        NEXUS_CRED = credentials('nexus-cred-id')
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/SrishtiC99/food-delivery-microservices.git'
            }
        }

        stage('Maven Build & Test') {
            steps {
                sh "mvn -B clean package"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                sh """
                    mvn sonar:sonar \
                        -Dsonar.projectKey=food-delivery \
                        -Dsonar.host.url=$SONAR_HOST \
                        -Dsonar.login=$SONAR_TOKEN
                """
            }
        }

        stage('Publish Artifacts to Nexus') {
            steps {
                sh "mvn deploy -Dnexus.url=$NEXUS_URL -Dnexus.user=$NEXUS_CRED_USR -Dnexus.pass=$NEXUS_CRED_PSW"
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    def services = ["user-service","order-service","payment-service","restaurant-service","delivery-service","auth-service","api-gateway","discovery-server"]

                    services.each { svc ->
                        sh """
                            cd ${svc}
                            docker build -t $REGISTRY/${svc}:${BUILD_NUMBER} .
                            docker tag  $REGISTRY/${svc}:${BUILD_NUMBER} $REGISTRY/${svc}:latest
                            cd ..
                        """
                    }
                }
            }
        }

        stage('Push to Docker Registry') {
            steps {
                script {
                    docker.withRegistry('', 'docker-credentials') {
                        def services = ["user-service","order-service","payment-service","restaurant-service","delivery-service","auth-service","api-gateway","discovery-server"]

                        services.each { svc ->
                            sh """
                                docker push $REGISTRY/${svc}:${BUILD_NUMBER}
                                docker push $REGISTRY/${svc}:latest
                            """
                        }
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh "kubectl apply -f k8s/"
            }
        }
    }
}
