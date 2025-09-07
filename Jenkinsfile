pipeline {
    agent any
    environment { 
        MAVEN_HOME = tool 'Maven 3'
        APP_USER = 'moreshraddha30'
        APP_VM_IP = '34.63.139.111'
        APP_DIR = '/home/moreshraddha30/app'
    }
    stages {

        stage('Checkout') {
            steps {
                echo "Checking out code from SCM..."
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                echo "Running Maven tests..."
                sh "${MAVEN_HOME}/bin/mvn -B test"
            }
        }

        stage('Package') {
            steps {
                echo "Packaging Java project..."
                sh "${MAVEN_HOME}/bin/mvn -B package"
                sh "ls -l target/"   // Debug: list .jar files
            }
        }

        stage('Deploy') {
            steps {
                sshagent(credentials: ['gce-ssh']) {
                    echo "Deploying .jar to App VM..."
                    sh '''
                        # Ensure app directory exists
                        ssh -o StrictHostKeyChecking=no $APP_USER@$APP_VM_IP "mkdir -p $APP_DIR"

                        # Copy .jar file
                        scp -o StrictHostKeyChecking=no target/*.jar $APP_USER@$APP_VM_IP:$APP_DIR/hello.jar

                        # Stop old app if running
                        ssh -o StrictHostKeyChecking=no $APP_USER@$APP_VM_IP \
                            'pkill -f "java -jar $APP_DIR/hello.jar" || true'

                        # Start new app
                        ssh -o StrictHostKeyChecking=no $APP_USER@$APP_VM_IP \
                            'nohup java -jar $APP_DIR/hello.jar > $APP_DIR/app.log 2>&1 &'
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Deployment completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check logs for details."
        }
    }
}
