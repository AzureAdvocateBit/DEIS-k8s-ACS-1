FROM java:8-jdk-alpine
MAINTAINER Yoshio Terada

VOLUME /tmp

ADD ./HelloWorld-1.0.jar /app.jar
RUN sh -c 'touch /app.jar'
ENV JAVA_OPTS=""

RUN chmod 755 /app.jar
EXPOSE 8080 8181
ENTRYPOINT java -jar app.jar --autoBindHttp --autoBindSsl 

