# 使用 Eclipse Temurin（AdoptOpenJDK 的继任者）
FROM eclipse-temurin:8-jre

# 创建并切换到工作目录
WORKDIR /app

# 从构建上下文（仓库根目录）复制 Jenkins 生成的 app.jar
COPY app.jar /app/app.jar

# 设置时区
ENV TZ=Asia/Shanghai

# 设置 JVM 参数（可通过 -e 覆盖）
ENV JAVA_OPTS="-Xms512m -Xmx512m -Djava.security.egd=file:/dev/./urandom"

# 应用自定义参数
ENV ARGS=""

# 暴露端口
EXPOSE 48080

# 启动应用
CMD ["sh", "-c", "java ${JAVA_OPTS} -jar /app/app.jar ${ARGS}"]