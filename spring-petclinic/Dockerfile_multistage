# syntax=docker/dockerfile:1

# Add the `base` label to allow us to refer to this build stage in other build stages.
FROM eclipse-temurin:17-jdk-jammy as base
WORKDIR /app
COPY .mvn/ .mvn
COPY mvnw pom.xml ./

# Resolve all test scope (includes compile) dependencies and then displays the resolved versions.
# - to help ensure all dependencies are downloaded to the local repository
# - can also be used to quickly determine how versions are being resolved
RUN ./mvnw dependency:resolve
COPY src ./src

FROM base as development
# Expose port 8000 and declare debug configuration for the JVM so that we can attach a debugger
CMD ["./mvnw", "spring-boot:run", "-Dspring-boot.run.profiles=mysql", "-Dspring-boot.run.jvmArguments='-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000'"]

FROM base as build
RUN ./mvnw package

FROM eclipse-temurin:17-jre-jammy as production
EXPOSE 8080
COPY --from=build /app/target/spring-petclinic-*.jar /spring-petclinic.jar
CMD ["java", "-Djava.security.egd=file:/dev/./urandom", "-jar", "/spring-petclinic.jar"]
