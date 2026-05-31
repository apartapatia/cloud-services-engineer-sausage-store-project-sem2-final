# =========================================
# Stage 1: Build
# =========================================

FROM maven:3.8.6-eclipse-temurin-17-alpine as builder

ARG VERSION=${VERSION}

WORKDIR /usr/src/app

COPY pom.xml ./

RUN mvn dependency:go-offline

COPY ./src ./src

RUN mvn package -Dversion.application=${VERSION}

# =========================================
# Stage 2: Runtime
# =========================================

FROM eclipse-temurin:17-jre-alpine

ARG VERSION=${VERSION}

WORKDIR /app

RUN addgroup -S apartgroup \
    && adduser -S -D -H -s /bin/false -G apartgroup apartuser \
    && apk add --no-cache dumb-init curl \
    && mkdir -p /logs/reports \
    && chown -R apartuser:apartgroup /logs

COPY --chown=apartuser:apartuser \
     --from=builder /usr/src/app/target/sausage-store-${VERSION}.jar ./sausage-store.jar

EXPOSE 8080

USER apartuser

ENTRYPOINT ["dumb-init", "java", "-jar", "-Dmyserver.bindPort=8080", "./sausage-store.jar"]