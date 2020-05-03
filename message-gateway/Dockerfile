FROM tomcat:7.0

ENV JAVA_OPTS="$JAVA_OPTS -DQUEUE_HOST=rabbit"

COPY ./target/message-gateway-1.0-SNAPSHOT.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080

CMD ["catalina.sh", "run"]