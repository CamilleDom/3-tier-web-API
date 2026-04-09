FROM eclipse-temurin:21-jdk-alpine AS builder
WORKDIR /opt/build

RUN apk add --no-cache maven

COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

FROM eclipse-temurin:21-jre-alpine
WORKDIR /opt/app

COPY --from=builder /opt/build/target/*.jar ./app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
