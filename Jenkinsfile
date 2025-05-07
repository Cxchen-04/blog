pipeline {
    agent any
    environment {
        GIT_REPO = 'https://github.com/Cxchen-04/blog.git'
        // GIT_REPO = 'git@github.com:Cxchen-04/blog.git'
        // DEPLOY_SERVER = '47.242.175.167'
        DEPLOY_SERVER = '8.218.189.240'
        DEPLOY_USER = 'root'
        // DEPLOY_PATH =  '/var/www/html'
        DEPLOY_PATH = '/root/blog'
        SSH_CREDENTIALS_ID = 'ecs-ssh'
        CREDENTIALS_ID = 'github-token'
        // LOCAL_HEXO_DIR = '/Users/macos/Desktop/blog'
    }
    stages {
        stage('从github上拉取代码'){
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: env.GIT_REPO,
                        credentialsId: env.CREDENTIALS_ID
                    ]]
                ])
                // git url: "${env.GIT_REPO}", branch: 'main'
            }
        }
        stage('清楚旧的文件'){
            steps {
                script() {
                    sshagent (credentials: ["${env.SSH_CREDENTIALS_ID}"]) {
                        sh """
                            ssh ${DEPLOY_USER}@${DEPLOY_SERVER} 'cd "${DEPLOY_PATH}" && rm -rf public'
                        """
                    }
                }
            }
        }
        stage('将public部署到ECS'){
            steps {
                script() {
                    sshagent (credentials: ["${env.SSH_CREDENTIALS_ID}"]) {
                        sh """
                            echo "🚀 正在部署 public 到服务器：${DEPLOY_SERVER}"
                            scp -r ./public ${DEPLOY_USER}@${DEPLOY_SERVER}:${DEPLOY_PATH}/
                        """
                    }
                }
            }
        }
        stage('重启docker-compose'){
            steps {
                script() {
                    sshagent (credentials: ["${env.SSH_CREDENTIALS_ID}"]) {
                        sh """
                            echo "🔄 重启 docker-compose"
                            ssh ${DEPLOY_USER}@${DEPLOY_SERVER} 'cd ${DEPLOY_PATH} && docker compose restart nginx'
                        """
                    }
                }
            }
        }
    }
}


        // stage('压缩public文件夹'){
        //     steps {
        //         sh """
        //             cd ${env.LOCAL_HEXO_DIR}
        //             zip -r ../public.zip ./public
        //         """
        //     }
        // }
        // stage('删除之前的public文件夹'){
        //     steps {
        //         script {
        //             sshagent (credentials: ["${env.SSH_CREDENTIALS_ID}"]) {
        //                 sh """
        //                     ssh ${DEPLOY_USER}@${DEPLOY_SERVER} 'cd ${DEPLOY_PATH} && rm -rf public'
        //                 """
        //             }
        //         }
        //     }
        // }
        // stage('上传并部署'){
        //     steps {
        //         script {
        //             sshagent (credentials: ["${env.SSH_CREDENTIALS_ID}"]) {
        //                 sh """
        //                     scp ${env.LOCAL_HEXO_DIR}/../public.zip ${DEPLOY_USER}@${DEPLOY_SERVER}:${DEPLOY_PATH}/
        //                     ssh ${DEPLOY_USER}@${DEPLOY_SERVER} 'cd ${DEPLOY_PATH} && unzip -o public.zip && rm -f public.zip'
        //                     ssh ${DEPLOY_USER}@${DEPLOY_SERVER} 'cd ${DEPLOY_PATH} && chmod -R 755 public'
        //                 """
        //             }
        //         }
        //     }
        // }
//     }
// }