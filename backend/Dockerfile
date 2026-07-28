FROM maven:3.9-eclipse-temurin-21 AS build

WORKDIR /workspace
COPY pom.xml .
RUN mvn -B dependency:go-offline

COPY src ./src
RUN mvn -B clean package -DskipTests

FROM eclipse-temurin:21-jre

RUN useradd --system --create-home --uid 10001 offerforge
WORKDIR /app
COPY --from=build /workspace/target/offerforge-backend-*.jar app.jar

USER offerforge
EXPOSE 8080

ENTRYPOINT ["java", "-jar", "/app/app.jar"]
