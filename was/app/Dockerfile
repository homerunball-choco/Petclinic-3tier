
# syntax=docker/dockerfile:1

# [전역 빌드 인자]
ARG BUILD_DIR=workspace
ARG APP_NAME=petclinic

# ============================================================
# STAGE 01: build (JDK 17)
# ============================================================
FROM eclipse-temurin:17-jdk AS build

ARG BUILD_DIR
ARG APP_NAME

WORKDIR /${BUILD_DIR}

# Maven 설정을 먼저 복사해 의존성 단계의 캐시 재사용
COPY .mvn/ ./.mvn/
COPY mvnw pom.xml ./

# Maven 의존성 다운로드 및 캐시
RUN --mount=type=cache,target=/root/.m2 \
    sh ./mvnw -B dependency:go-offline

# 애플리케이션 소스 복사
COPY src/ ./src/

# WAR 빌드
RUN --mount=type=cache,target=/root/.m2 \
    sh ./mvnw -B -DskipTests package

# ============================================================
# STAGE 02: runtime (JRE 17 + Alpine + Tomcat 9)
# ============================================================
FROM eclipse-temurin:17-jre-alpine AS runtime

ARG APP_NAME

ENV CATALINA_HOME=/opt/tomcat
ENV JAVA_TOOL_OPTIONS="-XX:MaxRAMPercentage=75.0"

# Tomcat 설치에 필요한 패키지
RUN apk add --no-cache curl ca-certificates && \
    mkdir -p /opt/tomcat && \
    curl -fSL \
      https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.123/bin/apache-tomcat-9.0.123.tar.gz \
      -o /tmp/tomcat.tar.gz && \
    tar -xzf /tmp/tomcat.tar.gz \
      -C /opt/tomcat --strip-components=1 && \
    rm /tmp/tomcat.tar.gz && \
    rm -rf /opt/tomcat/webapps/*

# 비root 사용자 생성
RUN addgroup -S -g 10001 petclinic && \
    adduser -S -D -u 10001 -G petclinic petclinic && \
    chown -R petclinic:petclinic /opt/tomcat

# 빌드 단계에서 생성한 WAR만 복사
COPY --from=build --chown=10001:10001 \
    /workspace/target/petclinic.war \
    /opt/tomcat/webapps/petclinic.war

# DB 접속 정보는 컨테이너 실행 시 환경변수로 전달
# DB_HOST, DB_PORT, DB_NAME, DB_USER, DB_PASSWORD

USER 10001:10001

EXPOSE 8080

ENTRYPOINT ["/opt/tomcat/bin/catalina.sh", "run"]
