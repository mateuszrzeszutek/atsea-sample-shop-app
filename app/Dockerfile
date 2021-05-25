FROM node:latest AS storefront
WORKDIR /usr/src/atsea/app/react-app
COPY react-app .
RUN npm install
RUN npm run build

FROM maven:latest AS appserver
WORKDIR /usr/src/atsea
COPY pom.xml .
RUN mvn -B -f pom.xml -s /usr/share/maven/ref/settings-docker.xml dependency:resolve
COPY . .
RUN mvn -B -s /usr/share/maven/ref/settings-docker.xml package -DskipTests

# download the latest splunk javaagent version as part of the build process
ADD https://github.com/signalfx/splunk-otel-java/releases/latest/download/splunk-otel-javaagent-all.jar splunk-otel-javaagent-all.jar
# add read permissions for everybody - the application will be run with user!=root
RUN chmod a+r splunk-otel-javaagent-all.jar

FROM java:8-jdk-alpine
RUN adduser -Dh /home/gordon gordon
WORKDIR /static
COPY --from=storefront /usr/src/atsea/app/react-app/build/ .
WORKDIR /app
COPY --from=appserver /usr/src/atsea/target/AtSea-0.0.1-SNAPSHOT.jar .
# copy the javaagent from the build step too
COPY --from=appserver /usr/src/atsea/splunk-otel-javaagent-all.jar .
# and add -javaagent to java command args
ENTRYPOINT ["java", "-javaagent:/app/splunk-otel-javaagent-all.jar", "-jar", "/app/AtSea-0.0.1-SNAPSHOT.jar"]
CMD ["--spring.profiles.active=postgres"]
