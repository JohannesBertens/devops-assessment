FROM openjdk:12-alpine

RUN mkdir -p /usr/src/app
COPY ./bin/airports-assembly-1.0.1.jar /usr/src/app
WORKDIR /usr/src/app

CMD ["java", "-jar", "airports-assembly-1.0.1.jar"]