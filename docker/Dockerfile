FROM opensuse
MAINTAINER Michael Englert <michi.eng@gmail.com>

ARG BASEURL
ARG MACHINE_AGENT_VERSION
ARG SAP_AGENT_VERSION
ARG USER
ARG PASSWORD

RUN zypper --non-interactive update \
    && zypper --non-interactive install -l curl unzip glibc-locale java-1_8_0-openjdk vim which \
    && curl --referer http://www.appdynamics.com -c /tmp/cookies.txt -d "username=$USER&password=$PASSWORD" https://login.appdynamics.com/sso/login/ \
    && curl -L -b /tmp/cookies.txt -o /tmp/machineagent.zip $BASEURL/machine/$MACHINE_AGENT_VERSION/MachineAgent-$MACHINE_AGENT_VERSION.zip \
    && curl -L -b /tmp/cookies.txt -o /tmp/sapagent.zip $BASEURL/sap-agent/$SAP_AGENT_VERSION/APPD-SAP-$SAP_AGENT_VERSION.zip \
    && mkdir -p /opt/appdynamics \
    && unzip /tmp/machineagent.zip -d /opt/appdynamics/machineagent \
    && unzip /tmp/sapagent.zip -d /opt/appdynamics/sapagent \
    && mv /opt/appdynamics/sapagent/APPD-SAP-*/SapAgent-*/appdhttpsdk/ /opt/appdynamics/appdhttpsdk/ \
    && chmod +x /opt/appdynamics/appdhttpsdk/* \
    && zypper --non-interactive -q rm -u curl unzip \
    && rm -rf /tmp/*.zip /opt/appdynamics/sapagent

EXPOSE 7999 8293 8900

CMD tail -f /dev/null
