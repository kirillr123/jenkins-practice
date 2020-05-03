FROM openjdk:7

COPY target/message-processor-1.0-SNAPSHOT.jar /usr/src/myapp.jar
WORKDIR /usr/src/

ENV QUEUE_HOST="messageGatewayQueue" 

CMD ["java", "-jar", "myapp.jar"]
