FROM openjdk:8-jdk-alpine

EXPOSE 80

RUN mkdir -p /var/log/twoja-app/ && chmod -R a+rw /var/log/twoja-app/ && mkdir /src/app/

WORKDIR /src/app/

ARG JAR_FILE=twoja-appka.jar

COPY ${JAR_FILE} app.jar

ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","app.jar"]
