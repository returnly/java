FROM alpine:3.8

USER root

RUN mkdir -p /deployments

# JAVA_APP_DIR is used by run-java.sh for finding the binaries
ENV JAVA_APP_DIR /deployments
ENV JAVA_HOME /usr/lib/jvm/default-jvm
ENV JAVA_OPTIONS "-XX:-OmitStackTraceInFastThrow -XX:+HeapDumpOnOutOfMemoryError"
ENV PATH $PATH:$JAVA_HOME/bin
# turn off agent bond by default
ENV AB_OFF true


# /dev/urandom is used as random source, which is prefectly safe according to http://www.2uo.de/myths-about-urandom/
RUN apk --no-cache update \
 && apk --no-cache add --update \
            curl \
            openjdk8 \
            bind-tools \
            jq \
            netcat-openbsd \
            groff less python py-pip ca-certificates \
 && pip --no-cache-dir install awscli boto3 \
 && apk --purge -v del py-pip \
 && rm -f /var/cache/apk/* \
 && echo "securerandom.source=file:/dev/urandom" >> /usr/lib/jvm/default-jvm/jre/lib/security/java.security

# Agent bond including Jolokia and jmx_exporter
ADD agent-bond-opts /opt/run-java-options
RUN mkdir -p /opt/agent-bond \
 && curl http://central.maven.org/maven2/io/fabric8/agent-bond-agent/1.0.2/agent-bond-agent-1.0.2.jar \
          -o /opt/agent-bond/agent-bond.jar \
 && chmod 444 /opt/agent-bond/agent-bond.jar \
 && chmod 755 /opt/run-java-options
ADD jmx_exporter_config.yml /opt/agent-bond/
EXPOSE 8080 8778 9779


# Add run script as /deployments/run-java.sh and make it executable
COPY run-java.sh debug-options container-limits java-default-options /deployments/
RUN chmod 755 /deployments/run-java.sh /deployments/java-default-options /deployments/container-limits /deployments/debug-options

# Grab SSM resolver python script used as the launcher and make it executable
RUN aws s3 cp s3://returnly-cfn-templates/bin/resolve-ssm-params.py /deployments/
RUN chmod 755 /deployments/resolve-ssm-params.py

RUN aws s3 cp s3://returnly-java-repo/releases/com/returnly/tools/java-tools/latest/java-tools-latest.jar /opt/java-tools.jar

RUN sed -i'' 's/#networkaddress.cache.ttl=-1/networkaddress.cache.ttl=30/' /usr/lib/jvm/java-1.8-openjdk/jre/lib/security/java.security

CMD [ "/deployments/resolve-ssm-params.py", "/deployments/run-java.sh" ]
