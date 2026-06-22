pipeline {
    agent any

    tools {
        maven 'Maven3'
    }

    options {
        timestamps()
    }

    environment {
        TOMCAT_USER = 'ec2-user'
        TOMCAT_HOST = 'http://3.94.201.155/'
        TOMCAT_HOME = '/home/ec2-user/apache'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Building WAR file...'
                sh 'mvn clean package'
            }
        }

        stage('Verify Artifact') {
            steps {
                echo 'Verifying generated WAR file...'
                sh 'ls -lh target/'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sh '''
                echo "Removing previous deployment..."

                ssh -o StrictHostKeyChecking=no ${TOMCAT_USER}@${TOMCAT_HOST} "
                    rm -rf ${TOMCAT_HOME}/webapps/MyApp
                    rm -f ${TOMCAT_HOME}/webapps/MyApp.war
                "

                echo "Copying WAR file to Tomcat server..."

                scp -o StrictHostKeyChecking=no \
                target/MyApp.war \
                ${TOMCAT_USER}@${TOMCAT_HOST}:${TOMCAT_HOME}/webapps/

                echo "Waiting for Tomcat auto deployment..."
                sleep 10
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                ssh -o StrictHostKeyChecking=no ${TOMCAT_USER}@${TOMCAT_HOST} "
                    echo '===== WEBAPPS DIRECTORY ====='
                    ls -lh ${TOMCAT_HOME}/webapps

                    echo ''
                    echo '===== VERIFY APPLICATION ====='
                    test -d ${TOMCAT_HOME}/webapps/MyApp
                "
                '''
            }
        }
    }

    post {

        success {
            echo '========================================'
            echo 'CI/CD PIPELINE COMPLETED SUCCESSFULLY'
            echo 'Application deployed to Tomcat Server'
            echo 'Tomcat Host: http://3.94.201.155'
            echo 'Application URL:'
            echo 'http://3.94.201.155/MyApp/'
            echo '========================================'
        }

        failure {
            echo '========================================'
            echo 'CI/CD PIPELINE FAILED'
            echo 'Check Console Output for errors.'
            echo '========================================'
        }

        always {
            cleanWs()
        }
    }
}
