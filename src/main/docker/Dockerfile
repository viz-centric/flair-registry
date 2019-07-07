FROM openjdk:12

LABEL maintainer="admin@vizcentric.com"
LABEL name="Vizcentric"

ENV SPRING_OUTPUT_ANSI_ENABLED=ALWAYS \
    JAVA_OPTS="" \
    JHIPSTER_SLEEP=0

VOLUME /tmp
EXPOSE 8761

ADD https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/v0.3.0/grpc_health_probe-linux-amd64 /bin/grpc_health_probe
COPY *.war /app.war

RUN chmod +x /bin/grpc_health_probe

CMD echo "The application will start in ${JHIPSTER_SLEEP}s..." && \
    sleep ${JHIPSTER_SLEEP} && \
    java ${JAVA_OPTS} -Djava.security.egd=file:/dev/./urandom -jar /app.war

