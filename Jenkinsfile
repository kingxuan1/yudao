pipeline {
    agent any

    environment {
        IMAGE_NAME = "yudao-backend"
        CONTAINER_NAME = "yudao-app"
    }

    stages {
        // 1. 验证工作区是否包含 pom.xml
        stage('Verify pom.xml') {
            steps {
                sh '''
                    echo "=== Current Directory ==="
                    pwd
                    echo "=== List Files in Workspace ==="
                    ls -la
                    echo "=== Check for pom.xml ==="
                    if [ -f pom.xml ]; then
                        echo "✅ pom.xml found!"
                    else
                        echo "❌ pom.xml NOT found!"
                        exit 1
                    fi
                '''
            }
        }

        // 2. 显式 checkout（可选，通常自动完成）
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // 3. 使用 Docker 中的 Maven 构建 JAR
        stage('Build JAR with Maven in Docker') {
            steps {
                sh 'docker run --rm -v "$PWD":/usr/src/mymaven -w /usr/src/mymaven maven:3.8.6-openjdk-8 mvn clean package -DskipTests'
                sh 'ls -l ./target/yudao-server.jar'
            }
        }

        // 4. 构建 Docker 镜像
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }

        // 5. 运行容器
        stage('Run Container') {
            steps {
                sh "docker stop ${CONTAINER_NAME} || true"
                sh "docker rm ${CONTAINER_NAME} || true"
                sh "docker run -d --name ${CONTAINER_NAME} -p 48080:48080 ${IMAGE_NAME}"
            }
        }

        // 6. 健康检查
        stage('Health Check') {
            steps {
                sh 'sleep 15'
                sh 'curl -f http://localhost:48080/admin-api/system/health || exit 1'
            }
        }
    }
}