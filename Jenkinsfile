pipeline {
    agent any

    tools {
        jdk 'Java17'
        maven 'Maven3'
    }

    environment {
        APP_NAME = 'microservice-app'
        STAGING_HOST = 'user@staging-server-ip'
        SONAR_ENV = 'SonarQube'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-ssh',
                    url: 'git@github.com:org/microservice-app.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Automated Tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv("${SONAR_ENV}") {
                    sh """
                    mvn sonar:sonar \
                    -Dsonar.projectKey=${APP_NAME} \
                    -Dsonar.projectName=${APP_NAME}
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }

        stage('Deploy to Staging') {
            steps {
                sshagent(credentials: ['staging-ssh']) {
                    sh """
                    scp target/${APP_NAME}.jar ${STAGING_HOST}:/opt/apps/
                    ssh ${STAGING_HOST} '
                        pkill -f ${APP_NAME}.jar || true
                        nohup java -jar /opt/apps/${APP_NAME}.jar > app.log 2>&1 &
                    '
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment to staging successful'
        }
        failure {
            echo 'Pipeline failed'
        }
    }
}
