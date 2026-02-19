pipeline {
    agent any

    environment {
        // Backend Config
        EC2_HOST = '65.0.185.151'
        PROJECT_DIR = '/home/ubuntu/foodapp/backend'

        // AWS / Frontend Config
        AWS_DEFAULT_REGION = 'ap-south-1'
        S3_BUCKET = 'foodapp99'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/abhinav-sharma-dev/foodapp.git'
            }
        }

        // =========================
        // BACKEND DEPLOYMENT
        // =========================
        stage('Deploy Backend to EC2') {
            steps {
                sshagent(['ec2-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@${EC2_HOST} '
                            cd ${PROJECT_DIR} || exit 1 &&
                            git pull origin main &&
                            npm install &&
                            pm2 describe foodapp > /dev/null 2>&1 &&
                            pm2 restart foodapp ||
                            pm2 start app.js --name foodapp --watch -- --port 5000
                        '
                    """
                }
            }
        }

        // =========================
        // FRONTEND BUILD
        // =========================
        stage('Install Frontend Dependencies') {
            steps {
                dir('frontend') {
                    sh 'npm install'
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir('frontend') {
                    sh 'npm run build'
                }
            }
        }

        // =========================
        // DEPLOY FRONTEND TO S3
        // =========================
        stage('Deploy Frontend to S3') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-s3-creds'
                ]]) {
                    sh '''
                        aws s3 sync frontend/build/ s3://$S3_BUCKET --delete
                    '''
                }
            }
        }
    }
}
