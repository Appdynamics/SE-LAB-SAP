FROM alpine AS builder
LABEL maintainer="Michael Englert <michi.eng@gmail.com>"

ARG BASEURL="https://download.appdynamics.com/download/prox/download-file"
ARG SAP_AGENT_VERSION
ARG USER
ARG PASSWORD

RUN apk update  
RUN apk upgrade
RUN apk add unzip curl
RUN mkdir -p /opt/appdynamics
RUN curl --referer http://www.appdynamics.com -c /tmp/cookies.txt -d "username=${USER}&password=${PASSWORD}" https://login.appdynamics.com/sso/login/
RUN curl -L -b /tmp/cookies.txt -o /tmp/sapagent.zip ${BASEURL}/sap-agent/${SAP_AGENT_VERSION}/APPD-SAP-${SAP_AGENT_VERSION}.zip
RUN unzip /tmp/sapagent.zip -d /opt/appdynamics/sapagent
RUN mv /opt/appdynamics/sapagent/APPD-SAP-*/SapAgent-*/appdhttpsdk/ /opt/appdynamics/appdhttpsdk/
RUN chmod +x /opt/appdynamics/appdhttpsdk/* 
RUN sed -i -e "s/\(.*\) nohup\(.*\)\(<&.*\)/\1\2#\3/" /opt/appdynamics/appdhttpsdk/runSDKManager
RUN sed -i -e "s/\(.*\) \([>][>]*[ ]*\/dev\/null.*\)/\1 #\2/g" /opt/appdynamics/appdhttpsdk/runSDKManager
RUN sed -i -e "s/#\!\/bin\/sh/#\!\/bin\/bash/" /opt/appdynamics/appdhttpsdk/runSDKManager
RUN sed -i -e "s/#\!\/bin\/sh/#\!\/bin\/bash/" /opt/appdynamics/appdhttpsdk/appd-http-sdk-proxy
RUN sed -i -e "s/#\!\/bin\/sh/#\!\/bin\/bash/" /opt/appdynamics/appdhttpsdk/runAppdHttpSDK

FROM openjdk:8-jre-slim
LABEL maintainer="Michael Englert <michi.eng@gmail.com>"

RUN apt-get update \
    && apt-get install --fix-missing -q -y procps

COPY --from=builder /opt/appdynamics/appdhttpsdk /opt/appdynamics/appdhttpsdk

EXPOSE 7999 8900

CMD [ "/bin/bash", "-c", "/opt/appdynamics/appdhttpsdk/runSDKManager" ]