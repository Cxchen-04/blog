pipeline {
    agent any
    environment {
        DEPLOY_SERVER = '47.242.175.167'
        DEPLOY_USER = 'root'
        DEPLOY_PATH =  '/var/www/html'
        SSH_CREDENTIALS_ID = 'ecs-ssh'
        LOCAL_HEXO_DIR = '/Users/macos/Desktop/blog'
    }
    stages {
        stage('压缩public文件夹'){
            steps {
                sh """
                    cd ${env.LOCAL_HEXO_DIR}
                    zip -r ../public.zip ./public
                """
            }
        }
        stage('删除之前的public文件夹'){
            steps {
                script {
                    sshagent (credentials: ["${env.SSH_CREDENTIALS_ID}"]) {
                        sh """
                            ssh ${DEPLOY_USER}@${DEPLOY_SERVER} 'cd ${DEPLOY_PATH} && rm -rf public'
                        """
                    }
                }
            }
        }
        stage('上传并部署'){
            steps {
                script {
                    sshagent (credentials: ["${env.SSH_CREDENTIALS_ID}"]) {
                        sh """
                            scp ${env.LOCAL_HEXO_DIR}/../public.zip ${DEPLOY_USER}@${DEPLOY_SERVER}:${DEPLOY_PATH}/
                            ssh ${DEPLOY_USER}@${DEPLOY_SERVER} 'cd ${DEPLOY_PATH} && unzip -o public.zip && rm -f public.zip'
                            ssh ${DEPLOY_USER}@${DEPLOY_SERVER} 'cd ${DEPLOY_PATH} && chmod -R 755 public'
                        """
                    }
                }
            }
        }
    }
}