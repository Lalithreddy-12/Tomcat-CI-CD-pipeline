pipeline {
    agent any

    tools {
        maven 'Maven3'
    }

    environment {
        TOMCAT_USER = 'ec2-user'
        TOMCAT_HOST = '172.31.91.169'
        TOMCAT_HOME = '/home/ec2-user/apache'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean clean package'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                echo "Deploying WAR to Tomcat..."

                ssh -o StrictHostKeyChecking=no ${TOMCAT_USER}@${TOMCAT_HOST} "
                    rm -rf ${TOMCAT_HOME}/webapps/MyApp
                    rm -f ${TOMCAT_HOME}/webapps/MyApp.war
                "

                scp -o StrictHostKeyChecking=no \
                target/MyApp.war \
                ${TOMCAT_USER}@${TOMCAT_HOST}:${TOMCAT_HOME}/webapps/

                echo "Deployment completed."
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                ssh -o StrictHostKeyChecking=no ${TOMCAT_USER}@${TOMCAT_HOST} "
                    ls -lh ${TOMCAT_HOME}/webapps
                "
                '''
            }
        }
    }

    post {
        success {
            echo '================================='
            echo ' CI/CD Pipeline Executed Successfully!'
            echo ' WAR File Built and Deployed'
            echo '================================='
        }

        failure {
            echo ' Pipeline Failed!'
        }
    }
}