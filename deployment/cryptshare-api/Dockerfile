FROM payara/server-full:4.181

ENV ADMIN_USER admin
ENV ADMIN_PASSWORD admin
ENV PAYARA_PATH /opt/payara41
ENV DOMAIN_NAME share
ENV PORT 7000


USER root
RUN apt-get update && apt-get install -y 
RUN wget https://dl.eff.org/certbot-auto
RUN chmod a+x certbot-auto
RUN ./certbot-auto --no-bootstrap

RUN echo 'AS_ADMIN_PASSWORD='${ADMIN_PASSWORD}'\n\
    EOF\n'\
    >> /opt/pwdfile

USER payara
RUN bin/asadmin --user ${ADMIN_USER} --passwordfile=/opt/pwdfile create-domain --portbase ${PORT} ${DOMAIN_NAME} 
COPY domain-template ${PAYARA_PATH}/glassfish/domains/share
COPY DSTRootCAX3.crt ${PAYARA_PATH}/glassfish/domains/share/config
COPY share-api.war ${PAYARA_PATH}/glassfish/domains/share/autodeploy



USER root
RUN chown -R payara:payara ${PAYARA_PATH}/glassfish/domains/share
RUN keytool -import -noprompt -alias tangleRoot -file ${PAYARA_PATH}/glassfish/domains/share/config/DSTRootCAX3.crt -keystore ${PAYARA_PATH}/glassfish/domains/share/config/cacerts.jks -storepass changeit

USER payara
RUN \
    bin/asadmin start-domain ${DOMAIN_NAME} && \
    bin/asadmin --host localhost --port 7048 --user ${ADMIN_USER} --passwordfile=/opt/pwdfile enable-secure-admin && \
    bin/asadmin restart-domain ${DOMAIN_NAME} 
EXPOSE 7080 7048

ENTRYPOINT bin/asadmin start-domain -v --debug ${DOMAIN_NAME}
