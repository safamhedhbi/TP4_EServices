FROM maven:3.6.0-jdk-8-alpine AS build
WORKDIR /usr/src/app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package -DskipTests

FROM openjdk:8-jre-alpine
COPY --from=build /usr/src/app/target/discovery-service-0.0.1-SNAPSHOT.jar /usr/app/
EXPOSE 8761
RUN apk add --no-cache bash
ADD wait-for-it.sh /wait-for-it.sh
CMD ["java","-jar","/usr/app/discovery-service-0.0.1-SNAPSHOT.jar","--spring.profiles.active=docker"]
