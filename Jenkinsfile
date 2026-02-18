pipeline {
    agent any

    tools {
        nodejs 'node25'
    }

    environment {
        DEPLOY_SERVER = 'linux@192.168.102.163'
        DEPLOY_PATH   = '/var/www/myapp'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Pulling source code...'
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build Artifact') {
            steps {
                sh 'tar --exclude=node_modules --exclude=.git -czf myapp.tar.gz .'
                archiveArtifacts artifacts: 'myapp.tar.gz', fingerprint: true
            }
        }

        stage('Deploy') {
            when {
                branch 'main'  // only deploy from main branch
            }
            steps {
                sshagent(['deploy-ssh-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${DEPLOY_SERVER} 'mkdir -p ${DEPLOY_PATH}'
                        scp myapp.tar.gz ${DEPLOY_SERVER}:${DEPLOY_PATH}/
                        ssh ${DEPLOY_SERVER} '
                            cd ${DEPLOY_PATH} &&
                            tar -xzf myapp.tar.gz &&
                            npm ci --production &&
                            pm2 restart myapp || pm2 start app.js --name myapp
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed — check the logs above.'
        }
    }
}
