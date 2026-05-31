# =========================================
# Stage 1: Build
# =========================================

FROM maven:3.8.6-eclipse-temurin-17-alpine AS builder

WORKDIR /usr/src/app

COPY pom.xml ./

RUN mvn dependency:go-offline

COPY ./src ./src

RUN mvn package -DskipTests \
    && cp target/sausage-store-*.jar target/app.jar

# =========================================
# Stage 2: Runtime
# =========================================

FROM eclipse-temurin:17-jre-alpine

WORKDIR /app

RUN addgroup -S apartgroup \
    && adduser -S -D -H -s /bin/false -G apartgroup apartuser \
    && apk add --no-cache dumb-init curl \
    && mkdir -p /logs/reports \
    && chown -R apartuser:apartgroup /logs

COPY --chown=apartuser:apartgroup \
     --from=builder /usr/src/app/target/app.jar ./sausage-store.jar

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=5s --start-period=30s --retries=3 \
  CMD curl -sf http://127.0.0.1:8080/actuator/health || exit 1

USER apartuser

ENTRYPOINT ["dumb-init", "java", "-jar", "-Dmyserver.bindPort=8080", "./sausage-store.jar"]
