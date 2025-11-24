pipeline {
    agent any

    environment {
        HARBOR_HOST     = '220.182.11.205:8086'
        HARBOR_PROJECT  = 'yudao'
        IMAGE_NAME      = 'yudao-backend'
        FULL_IMAGE_NAME = "${HARBOR_HOST}/${HARBOR_PROJECT}/${IMAGE_NAME}:latest"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build JAR') {
            steps {
                script {
                    def m2Path = "${WORKSPACE}/../m2"
                    docker.image('maven:3.8.6-openjdk-8').inside("-v ${m2Path}:/root/.m2") {
                        sh 'mvn clean package -DskipTests'
                        sh 'cp yudao-server/target/yudao-server.jar ./app.jar'
                    }
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'harbor-registry',
                        usernameVariable: 'HARBOR_USER',
                        passwordVariable: 'HARBOR_PASS'
                    )]) {
                        sh "docker build -f yudao-server/Dockerfile -t ${FULL_IMAGE_NAME} ."
                        sh "echo ${HARBOR_PASS} | docker login ${HARBOR_HOST} -u ${HARBOR_USER} --password-stdin"
                        sh "docker push ${FULL_IMAGE_NAME}"
                        sh "docker logout ${HARBOR_HOST}"
                    }
                }
            }
        }

        stage('Test Pull') {
            steps {
                sh '''
                    ssh SupUsr@220.182.11.205 '
                        cd /home/SupUsr/yudao &&
                        docker compose pull
                    '
                '''
            }
        }

        stage('Pull Image Only') {
            steps {
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: 'lz服务器',
                            transfers: [
                                sshTransfer(
                                    sourceFiles: 'lombok.config',  // ← 必须存在
                                    remoteDirectory: '',       // 可选
                                    execCommand: '''
                                        cd /home/SupUsr/yudao
                                        docker compose pull
                                    '''
                                )
                            ]
                        )
                    ]
                )
            }
        }
    }

    post {
        success { echo "✅ 镜像已成功推送至 Harbor: ${FULL_IMAGE_NAME}" }
        failure { echo "❌ 构建或推送失败1" }
    }
}