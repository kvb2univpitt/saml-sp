# saml-sp
A simple Spring Boot application for demonstrating SAML 2.0 service provider (SP).  This sample application can be run as is without any configuration.  However, you will need to run the docker SAML 2.0 identity provider (IdP) for the service provide to communicate for federated login (see instruction below).

 ## Prerequisites
 You will need the download and install following to compile and build the software:
 - [Java SE Development Kit 8](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html) or [OpenJDK 8](https://adoptopenjdk.net/?variant=openjdk8&jvmVariant=hotspot)
 - [Apache Maven 3.x.x](https://maven.apache.org/download.cgi)

You will need to install the [Docker](https://docs.docker.com/get-docker/) to run the docker SAML 2.0 IdP.

## Run the docker SAML 2.0 IdP
The docker image for the SAML 2.0 IdP is in the DockerHub Registry.  To pull the image from DockerHub and run the container, execute the following command:
```
docker run -d --name=testsamlidp_idp \
-p 8080:8080 \
-p 8443:8443 \
-e SIMPLESAMLPHP_SP_ENTITY_ID=https://localhost:6443/saml2/service-provider-metadata/samltestidp \
-e SIMPLESAMLPHP_SP_ASSERTION_CONSUMER_SERVICE=https://localhost:6443/login/saml2/sso/samltestidp \
-e SIMPLESAMLPHP_SP_SINGLE_LOGOUT_SERVICE=https://localhost:6443/logout \
-e SIMPLESAMLPHP_UID=1 \
-e SIMPLESAMLPHP_USER=jkim \
-e SIMPLESAMLPHP_PASSWORD=inyourarea \
-e SIMPLESAMLPHP_GROUP=blackpink \
-e SIMPLESAMLPHP_EMAIL=jkim@yg.com \
-e SIMPLESAMLPHP_FIRST_NAME=Jennie \
-e SIMPLESAMLPHP_LAST_NAME=Kim \
kvb2univpitt/test-saml2-idp:v1
```
