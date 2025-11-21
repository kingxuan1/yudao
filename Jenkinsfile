pipeline {
    agent any

    environment {
        IMAGE_NAME = "yudao-backend"
        CONTAINER_NAME = "yudao-app"
    }

    stages {
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

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build JAR with Maven in Docker') {
            steps {
                script {
                    // 复用 docker-compose.yml 中定义的 ./m2 目录
                    // Jenkins 容器内工作区: /var/jenkins_home/workspace/...
                    // 主机项目结构: ./m2 与 ./jenkins_home 同级
                    def m2Path = "${WORKSPACE}/../m2"
                    docker.image('maven:3.8.6-openjdk-8').inside("-v ${m2Path}:/root/.m2") {
                        sh 'mvn --version'
                        sh 'mvn clean package -DskipTests'
                        sh '''
                            echo "=== JAR Files in yudao-server/target/ ==="
                            ls -l yudao-server/target/
                            cp yudao-server/target/yudao-server.jar ./app.jar
                            ls -l ./app.jar
                        '''
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -f yudao-server/Dockerfile -t ${IMAGE_NAME} ."
            }
        }

        stage('Run Container') {
            steps {
                sh "docker stop ${CONTAINER_NAME} || true"
                sh "docker rm ${CONTAINER_NAME} || true"
                sh """
                    docker run -d \\
                      --name ${CONTAINER_NAME} \\
                      -p 48080:48080 \\
                      -e SPRING_PROFILES_ACTIVE=dev \\
                      ${IMAGE_NAME}
                """
            }
        }

        stage('Health Check') {
            steps {
                sh 'sleep 15'
                sh 'curl -f http://localhost:48080/admin-api/system/health || exit 1'
            }
        }
    }

    post {
        always {
            echo "Pipeline completed."
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}