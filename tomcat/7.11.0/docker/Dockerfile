# Builder Image
FROM alpine:3.10 as builder

ARG VERSION=7.11.0
ARG DISTRO=tomcat

RUN apk add --no-cache \
        ca-certificates \
        maven \
        tar \
        wget \
        git \
        && cd /tmp \
        && git clone https://github.com/iceman91176/camunda-json-logging.git \
        && cd camunda-json-logging/$DISTRO/$VERSION/json-logging-dependency-package \
        && mvn package \
        && cp target/camunda-json-logging-$DISTRO-$VERSION.jar /tmp

# Pull upstream CamundaBPM Tomcat image
FROM camunda/camunda-bpm-platform:tomcat-7.11.0

ARG VERSION=7.11.0
ARG DISTRO=tomcat

ENV PRETTY_JSON_LOG=false

# Remove the slf4j-jdj14 bridge library that is packaged with CamundaBPM tomcat distribution
# Add logback configuration dir
RUN rm -rf /camunda/lib/slf4j-jdk14-1.7*.jar \
    && mkdir /camunda/conf/logback

# Add logback configuration
COPY docker/camunda/conf/logback/logback.xml /camunda/conf/logback/logback.xml

# Overwrite global logging configuration for tomcat
COPY docker/camunda/conf/logging.properties /camunda/conf/logging.properties

# Copy Jar for logging
COPY --from=builder /tmp/camunda-json-logging-$DISTRO-$VERSION.jar /camunda/lib/
#COPY docker/camunda/lib/ /camunda/lib/

# Overwrite setenv.sh with additional CLASSPATH config that exposes the libraries at tomcat startup
COPY docker/camunda/bin/setenv.sh /camunda/bin/setenv.sh
