FROM ibmjava:8-jre as staging
  
# Install unzip; needed to unzip Open Liberty
RUN apt-get update \
    && apt-get install -y --no-install-recommends unzip \
    && rm -rf /var/lib/apt/lists/*

# Install Open Liberty
ENV LIBERTY_SHA 127281fdae81670ae20d87c92f8353f179da6c0e 
ENV LIBERTY_URL https://public.dhe.ibm.com/ibmdl/export/pub/software/openliberty/runtime/nightly/2018-05-14_1451/openliberty-all-20180514-1300.zip

RUN wget -q "$LIBERTY_URL" -U UA-Open-Liberty-Docker -O /tmp/wlp.zip \
   && echo "$LIBERTY_SHA  /tmp/wlp.zip" > /tmp/wlp.zip.sha1 \
   && sha1sum -c /tmp/wlp.zip.sha1 \
   && unzip -q /tmp/wlp.zip -d /opt/ol \
   && rm /tmp/wlp.zip \
   && rm /tmp/wlp.zip.sha1

# Prime server cache
COPY springServer/server.xml /opt/ol/wlp/usr/servers/springServer/server.xml
RUN /opt/ol/wlp/bin/server start springServer \
  && /opt/ol/wlp/bin/server stop springServer 

# Stage the fat JAR
ARG APP_FILE
COPY ${APP_FILE} /staging/myFatApp.jar

# Add the Open Liberty server
COPY springServer/appconfig.xml /opt/ol/wlp/usr/servers/springServer/appconfig.xml

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

COPY --from=staging /staging/myThinApp.jar /opt/ol/wlp/usr/servers/springServer/apps/myThinApp.jar

CMD ["/opt/ol/wlp/bin/server", "run", "springServer"]

