pipeline {
    agent any

    environment {
        SONARQUBE_ENV = 'SonarQube' // Jenkins > Configure System > SonarQube server
        MAVEN_HOME = tool 'Maven 3.9.11'
        EC2_USER = 'ubuntu'
        EC2_HOST = '54.175.161.255'
        EC2_SSH_CRED_ID = 'aws-ssh-key' // Jenkins Credentials ID
        SONAR_TOKEN = credentials('ubuntu')

    }

    stages {

        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/AnkitaJaiswal-git/spring-petclinic.git', branch: 'Rupesh'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh """
                        ${MAVEN_HOME}/bin/mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=spring-petclinic \
                        -Dsonar.login=$SONAR_TOKEN

                        # Simulated check — you should replace this with actual logic if needed
                        echo "BUILD SUCCESS" > status.txt
                    """
                }
            }
        }

        stage('Verify Condition to Proceed') {
            steps {
                script {
                    def output = sh(script: "cat status.txt", returnStdout: true).trim()
                    if (output != "BUILD_READY") {
                        error("Sonar analysis result not approved. Stopping pipeline.")
                    } else {
                        echo "Build condition met. Proceeding to next stage."
                    }
                }
            }
        }

        stage('Build Project') {
            steps {
                sh "${MAVEN_HOME}/bin/mvn package -DskipTests"
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent (credentials: ["${EC2_SSH_CRED_ID}"]) {
                    sh """
                        scp target/*.jar ${EC2_USER}@${EC2_HOST}:/home/${EC2_USER}/
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline executed successfully!'
        }
        failure {
            echo '❌ Pipeline failed due to an error or failed condition.'
        }
    }
}
