FROM ibmjava:8-jre as staging
  
# Install unzip; needed to unzip Open Liberty
RUN apt-get update \
    && apt-get install -y --no-install-recommends unzip \
    && rm -rf /var/lib/apt/lists/*

# Install Open Liberty
ENV LIBERTY_SHA 127281fdae81670ae20d87c92f8353f179da6c0e 
ENV LIBERTY_URL https://public.dhe.ibm.com/ibmdl/export/pub/software/openliberty/runtime/nightly/2018-05-14_1451/openliberty-all-20180514-1300.zip

ARG SPRING_BOOT_VERSION
RUN test -n "$SPRING_BOOT_VERSION"
ENV SPRING_BOOT_VERSION $SPRING_BOOT_VERSION

RUN wget -q "$LIBERTY_URL" -U UA-Open-Liberty-Docker -O /tmp/wlp.zip \
   && echo "$LIBERTY_SHA  /tmp/wlp.zip" > /tmp/wlp.zip.sha1 \
   && sha1sum -c /tmp/wlp.zip.sha1 \
   && unzip -q /tmp/wlp.zip -d /opt/ol \
   && rm /tmp/wlp.zip \
   && rm /tmp/wlp.zip.sha1 \
   && mkdir -p /opt/ol/wlp/usr/servers/springServer/ \
   && echo spring.boot.version="$SPRING_BOOT_VERSION" > /opt/ol/wlp/usr/servers/springServer/bootstrap.properties \
   && echo \
'<?xml version="1.0" encoding="UTF-8"?> \
<server description="Spring Boot Server"> \
  <featureManager> \
    <feature>jsp-2.3</feature> \
    <feature>transportSecurity-1.0</feature> \
    <feature>websocket-1.1</feature> \
    <feature>springBoot-${spring.boot.version}</feature> \
  </featureManager> \
  <httpEndpoint id="defaultHttpEndpoint" host="*" httpPort="9080" httpsPort="9443" /> \
  <include location="appconfig.xml"/> \
</server>' > /opt/ol/wlp/usr/servers/springServer/server.xml \
   && /opt/ol/wlp/bin/server start springServer \
   && /opt/ol/wlp/bin/server stop springServer \
   && echo \
'<?xml version="1.0" encoding="UTF-8"?> \
<server description="Spring Boot application config"> \
  <springBootApplication location="app.jar" name="Spring Boot application" /> \
</server>' > /opt/ol/wlp/usr/servers/springServer/appconfig.xml

# Stage the fat JAR
ARG APP_FILE
RUN test -n "$APP_FILE"
COPY ${APP_FILE} /staging/myFatApp.jar

# Thin the fat application; stage the thin app output and the library cache
RUN /opt/ol/wlp/bin/springBootUtility thin --sourceAppPath=/staging/myFatApp.jar --targetThinAppPath=/staging/myThinApp.jar --targetLibCachePath=/staging/lib.index.cache

# Put the final image together, only copying from staging:
#  The Open Liberty install
#  The library cache
#  The thin app
FROM ibmjava:8-jre

EXPOSE 9080

COPY --from=staging /opt/ol/wlp /opt/ol/wlp

COPY --from=staging /staging/lib.index.cache /opt/ol/wlp/usr/shared/resources/lib.index.cache

COPY --from=staging /staging/myThinApp.jar /opt/ol/wlp/usr/servers/springServer/apps/app.jar

CMD ["/opt/ol/wlp/bin/server", "run", "springServer"]

