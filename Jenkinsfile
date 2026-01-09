pipeline {
    agent any

    tools {
        maven 'MAVEN'
        jdk 'java17'
    }

    environment {
        TOMCAT_URL = "http://4.155.175.138:8081/"
        APP_NAME   = "simple-webapp"
        WAR_FILE   = "target/simple-webapp.war"
    }

    triggers {
        githubPush()
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Checking out source code"
                git branch: 'master',
                    url: 'https://github.com/cloudtech-trainer/simple-webapp.git'
            }
        }

        stage('Build') {
            steps {
                echo "Building WAR using Maven"
                sh 'mvn clean package'
            }
        }

        stage('Unit Tests') {
            steps {
                echo "Running unit tests"
                sh 'mvn test'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'tomcat-creds',
                    usernameVariable: 'TC_USER',
                    passwordVariable: 'TC_PASS'
                )]) {

                    sh """
                    echo "Undeploying existing application (if exists)"
                    curl -u $TC_USER:$TC_PASS \
                      "$TOMCAT_URL/manager/text/undeploy?path=/$APP_NAME" || true

                    echo "Deploying WAR to Tomcat"
                    curl -u $TC_USER:$TC_PASS \
                      -T $WAR_FILE \
                      "$TOMCAT_URL/manager/text/deploy?path=/$APP_NAME&update=true"
                    """
                }
            }
        }
    }

    post {
        success {
            echo "CI/CD Pipeline completed successfully"
            echo "Application URL: $TOMCAT_URL/$APP_NAME"
        }
        failure {
            echo "CI/CD Pipeline failed"
        }
    }
}
