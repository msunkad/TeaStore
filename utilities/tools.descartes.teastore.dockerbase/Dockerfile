FROM tomcat:8.5-jre8
MAINTAINER Chair of Software Engineering <se2-it@informatik.uni-wuerzburg.de>

# Delete all example webapps from base container
RUN rm -r /usr/local/tomcat/webapps/examples || true
RUN rm -r /usr/local/tomcat/webapps/docs || true
RUN rm -r /usr/local/tomcat/webapps/host-manager || true
RUN rm -r /usr/local/tomcat/webapps/manager || true

RUN rm /usr/local/tomcat/lib/websocket-api.jar || true
RUN rm /usr/local/tomcat/lib/tomcat-websocket.jar || true

# Copy config, database connector library and script for environment variables into container
COPY baseContext.xml /usr/local/tomcat/conf/context.xml
COPY mysql-connector-java-5.1.44-bin.jar /usr/local/tomcat/lib/mysql-connector-java-5.1.44-bin.jar
COPY target/jars/dockermemoryconfigurator* /usr/local/tomcat/bin/dockermemoryconfigurator.jar
RUN rm /usr/local/tomcat/conf/server.xml
COPY server.xml /usr/local/tomcat/conf/
COPY ssl/ /usr/local/tomcat/ssl


# Service configuration. SERVICE_PORT is the host port, mapped to the exposed docker port. HOST_NAME
# is the IP address or hostname of the host running the docker container
ENV SERVICE_PORT 8080
ENV HOST_NAME unset
ENV USE_POD_IP false
ENV REGISTRY_HOST 127.0.0.1
ENV REGISTRY_PORT 8080
ENV DB_HOST 127.0.0.1
ENV DB_PORT 3306
ENV RECOMMENDER_RETRAIN_LOOP_TIME 0
ENV RECOMMENDER_ALGORITHM SlopeOne
ENV PROXY_NAME unset
ENV PROXY_PORT unset
ENV TOMCAT_HEAP_MEM_PERCENTAGE 50
ENV LOG_TO_FILE false
ENV RABBITMQ_HOST unset
ENV USE_HTTPS false

# Copy script to replace placeholders in context.xml with the environment variables
COPY start.sh /usr/local/tomcat/bin/start.sh
RUN chmod +x /usr/local/tomcat/bin/start.sh

# Setting secure random number generation to urandom. This increases the startup speed.
# Environment variable no longer needed. Replaced by sed in start.sh
# ENV JAVA_OPTS $JAVA_OPTS -Dsecurerandom.source=file 837:/dev/urandom

RUN mkdir -p /kieker/logs
# Import self-signed certificate for HTTPS teastore into keystore. NOTE: to adapt certificate hostnames, configure and run ssl/generate_cert.sh and rebuild the container!
RUN keytool -import -noprompt -trustcacerts -alias teastoressl -file /usr/local/tomcat/ssl/cert.pem -keystore "$JAVA_HOME/lib/security/cacerts" -storepass changeit

COPY kieker.monitoring.properties 							/kieker/config/kieker.monitoring.properties
COPY aop.xml 												/usr/local/tomcat/lib/aop.xml
COPY kieker-1.14-SNAPSHOT-aspectj.jar 						/kieker/agent/agent.jar

EXPOSE 8080

CMD java -jar /usr/local/tomcat/bin/dockermemoryconfigurator.jar ${TOMCAT_HEAP_MEM_PERCENTAGE}; /usr/local/tomcat/bin/start.sh && /usr/local/tomcat/bin/catalina.sh run
