pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "santhoshdocker27/ekart:latest"   // Replace with your Docker Hub username & repo
    }

    tools {
        maven 'maven3'           // Use your Maven version (configure this in Jenkins tools)
        jdk 'Java21'             // Use your installed JDK (configure this in Jenkins tools)
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Santhoshdevopsgit/Ekart.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            environment {
                SONARQUBE_SCANNER_HOME = tool 'SonarQubeScanner'  // Use your configured SonarQube scanner name
            }
            steps {
                withSonarQubeEnv('MySonarQubeServer') {  // Replace with your SonarQube server name in Jenkins
                    sh "${SONARQUBE_SCANNER_HOME}/bin/sonar-scanner \
                        -Dsoanr.sonnar.url=https://   \
                        -Dsonar.login=
                        -Dsonar.projectKey=ekart \
                        -Dsonar.sources=. \
                        -Dsonar.java.binaries=target"
                }
            }
        }

        stage('OWASP Dependency-Check') {
            steps {
                sh 'mvn org.owasp:dependency-check-maven:check'
            }
        }

        stage('Build Application') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t $DOCKER_IMAGE .'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh 'docker push $DOCKER_IMAGE'
                    }
                }
            }
        }

        stage('Deploy to Docker') {
            steps {
                script {
                    // Stop existing container if running
                    sh '''
                    docker stop ekart-app || true
                    docker rm ekart-app || true
                    docker run -d --name ekart-app -p 8080:8080 $DOCKER_IMAGE
                    '''
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
