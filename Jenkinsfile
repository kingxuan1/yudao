pipeline {
    agent any

    triggers {
        GenericTrigger(
            genericVariables: [
                [key: 'ref', value: '$.ref']
            ],
            causeString: 'Triggered by GitHub push',
            token: 'yudao-github-auto',  // ← 自定义 token，记住它！
            printContributedVariables: true,
            printPostContent: false,
            silentResponse: false
        )
    }

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

        stage('Pull Image Only') {
            steps {
                sshagent(['lz-server-key']) {
                    sh '''
                        set -e  # 遇到错误立即退出
                        ssh -o StrictHostKeyChecking=no \
                            -p 3925 \
                            SupUsr@220.182.11.205 \
                            "cd /opt/yudao && /usr/bin/docker compose pull && /usr/bin/docker compose up -d"
                        echo "✅ 镜像拉取成功！"
                    '''
                }
            }
        }

    }

     post {
            success {
                echo "✅ 镜像已成功推送至 Harbor: ${FULL_IMAGE_NAME}"
                emailext (
                    subject: "✅ [SUCCESS] Pipeline 成功: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                    body: """
                        构建成功！
                        - 项目: ${env.JOB_NAME}
                        - 构建号: ${env.BUILD_NUMBER}
                        - Git 分支: ${env.GIT_BRANCH}
                        - 镜像: ${FULL_IMAGE_NAME}
                        - 查看报告: ${env.BUILD_URL}
                    """,
                    recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']],
                    to: '2755395209@qq.com'  // ← 替换成你的邮箱
                )
            }
            failure {
                echo "❌ 构建或推送失败"
                emailext (
                    subject: "❌ [FAILED] Pipeline 失败: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                    body: """
                        构建失败！
                        - 项目: ${env.JOB_NAME}
                        - 构建号: ${env.BUILD_NUMBER}
                        - Git 分支: ${env.GIT_BRANCH}
                        - 失败阶段: ${env.STAGE_NAME}
                        - 查看日志: ${env.BUILD_URL}console
                    """,
                    recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']],
                    to: '2755395209@qq.com'  // ← 替换成你的邮箱
                )
            }
     }
}