pipeline {
    agent any

    environment {
        IMAGE_NAME = "yudao-backend"
        CONTAINER_NAME = "yudao-app"
    }

    stage('Verify pom.xml') {
        steps {
            sh '''
                echo "=== Current Directory ==="
                pwd
                echo "=== List Files ==="
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

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build JAR with Maven in Docker') {
            steps {
                // 使用 maven:3.8.6-openjdk-8 镜像构建，自带 JDK 8 + Maven
                sh 'docker run --rm -v "$PWD":/usr/src/mymaven -w /usr/src/mymaven maven:3.8.6-openjdk-8 mvn clean package -DskipTests'
                sh 'ls -l ./target/yudao-server.jar'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }

        stage('Run Container') {
            steps {
                sh "docker stop ${CONTAINER_NAME} || true"
                sh "docker rm ${CONTAINER_NAME} || true"
                sh "docker run -d --name ${CONTAINER_NAME} -p 48080:48080 ${IMAGE_NAME}"
            }
        }

        stage('Health Check') {
            steps {
                sh 'sleep 15'
                sh 'curl -f http://localhost:48080/admin-api/system/health || exit 1'
            }
        }
    }
}