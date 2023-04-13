FROM eclipse-temurin:17-jre as builder
WORKDIR application
ARG JAR_FILE=application/jar/*.jar
COPY ${JAR_FILE} application.jar
RUN java -Djarmode=layertools -jar application.jar extract

################################

FROM eclipse-temurin:17-jre
WORKDIR application
COPY --from=builder application/dependencies/ ./
COPY --from=builder application/spring-boot-loader/ ./
COPY --from=builder application/snapshot-dependencies/ ./
COPY --from=builder application/application/ ./

ENV JVM_OPTS="-Xmx256m -Xms256m" \
    HALO_WORK_DIR="/root/.halo2" \
    SPRING_CONFIG_LOCATION="optional:classpath:/;optional:file:/root/.halo2/" \
    TZ=Asia/Shanghai
ENV MYSQL_HOST="localhost"
ENV MYSQL_PORT=3306
ENV MYSQL_PLATFORM="mysql"
ENV MYSQL_USERNAME="root"
ENV MYSQL_PASSWORD="root"
ENV HALO_EXTERNAL_URL="http://localhost:8090/"
ENV SUPER_ADMIN_USERNAME="admin"
ENV SUPER_ADMIN_PASSWORD="admin"

RUN ln -sf /usr/share/zoneinfo/$TZ /etc/localtime \
    && echo $TZ > /etc/timezone
RUN echo "MYSQL_HOST: ${MYSQL_HOST}"
EXPOSE 8090

ENTRYPOINT ["sh", "-c", "java ${JVM_OPTS} \
 -Dspring.r2dbc.url=r2dbc:pool:mysql://${MYSQL_HOST}:${MYSQL_PORT}/halo \
 -Dspring.r2dbc.username=${MYSQL_USERNAME} \
 -Dspring.r2dbc.password=${MYSQL_PASSWORD} \
 -Dspring.sql.init.platform=mysql=${MYSQL_PLATFORM} \
 -Dhalo.external-url=${HALO_EXTERNAL_URL} \
 -Dhalo.security.initializer.superadminusername=${SUPER_ADMIN_USERNAME} \
 -Dhalo.security.initializer.superadminpassword=${SUPER_ADMIN_PASSWORD} \
 org.springframework.boot.loader.JarLauncher ${0} ${@}"]
