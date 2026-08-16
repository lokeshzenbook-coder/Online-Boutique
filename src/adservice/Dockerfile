# Build stage
FROM eclipse-temurin:21-jdk-noble AS builder

WORKDIR /app

COPY gradle gradle
COPY gradlew build.gradle settings.gradle ./
RUN chmod +x gradlew

COPY src src

RUN ./gradlew installDist --no-daemon

# Runtime stage
FROM eclipse-temurin:21-jre-alpine

WORKDIR /app
COPY --from=builder /app/build/install/hipstershop ./

EXPOSE 9555

ENTRYPOINT ["/app/bin/AdService"]
