# saml-sp
A simple Spring Boot application for demonstrating SAML 2.0 service provider (SP).  This sample application can be run as is without any configuration.  However, you will need to run the docker SAML 2.0 identity provider (IdP) for the service provide to communicate for federated login (see instruction below).

To learn about how SAML works, please read the guide from Okta [https://developer.okta.com/docs/concepts/saml/](https://developer.okta.com/docs/concepts/saml/)

 ## Prerequisites
 You will need the download and install following to compile and build the software:
 - Java SDK 8 or higher ([Oracle JDK](https://www.oracle.com/java/technologies/javase-downloads.html) or [OpenJDK](https://adoptopenjdk.net/))
 - [Apache Maven 3.x.x](https://maven.apache.org/download.cgi)

You will need to install the [Docker](https://docs.docker.com/get-docker/) to run the docker SAML 2.0 IdP.

## Run the docker SAML 2.0 Identity Provider (IdP)
The first thing we want to do is to run the SAML identity service.  This service provides the ability to authenticate a user and to provide user profile such as first name, last name, email, group, etc to the service provider.

We provide a preconfigured SAML IdP docker image that works with this SAML service provider.  The docker image is in the DockerHub Registry.  Before we pull the dock image and run it, let's go over the input environment variables that will be needed.

### Input Environment Variables
Below is the table of all the variables that are required to run the SAML IdP.

| Variable | Type | Description |
|---|---|---|
| SIMPLESAMLPHP_SP_ENTITY_ID | Service Provider | Service provider entity ID |
| SIMPLESAMLPHP_SP_ASSERTION_CONSUMER_SERVICE | Service Provider | Service provider assertion consumer service location |
| SIMPLESAMLPHP_SP_SINGLE_LOGOUT_SERVICE | Service Provider | Service provider single logout service location |
| SIMPLESAMLPHP_ADMIN_PASSWORD | Admin Credential | Admin password |
| SIMPLESAMLPHP_USER | User Credential | User's login username |
| SIMPLESAMLPHP_PASSWORD | User Credential | User's login password |
| SIMPLESAMLPHP_UID | User Profile | Unique user ID |
| SIMPLESAMLPHP_GROUP | User Profile | User's group |
| SIMPLESAMLPHP_EMAIL | User Profile | User's email |
| SIMPLESAMLPHP_FIRST_NAME | User Profile | User's first name |
| SIMPLESAMLPHP_LAST_NAME | User Profile | User's last name |

### Example
Assuming we want to run a SAML IdP with the admin password ***letmein*** and the following user credential and profile information:

| UID | Username | Password | Group | Email | FirstName | LastName |
|---|---|---|---|---|---|---|
| 1 | ckent | batman | Daily Planet | ckent@dailyplanet.com | Clark | Kent |

Open up a terminal and execute the following command:

```
docker run -d --name=testsamlidp_idp \
-p 8080:8080 \
-p 8443:8443 \
-e SIMPLESAMLPHP_ADMIN_PASSWORD=letmein \
-e SIMPLESAMLPHP_SP_ENTITY_ID=https://localhost:6443/saml2/service-provider-metadata/samltestidp \
-e SIMPLESAMLPHP_SP_ASSERTION_CONSUMER_SERVICE=https://localhost:6443/login/saml2/sso/samltestidp \
-e SIMPLESAMLPHP_SP_SINGLE_LOGOUT_SERVICE=https://localhost:6443/logout \
-e SIMPLESAMLPHP_UID=1 \
-e SIMPLESAMLPHP_USER=ckent \
-e SIMPLESAMLPHP_PASSWORD=batman \
-e SIMPLESAMLPHP_GROUP="Daily Planet" \
-e SIMPLESAMLPHP_EMAIL=ckent@dailyplanet.com \
-e SIMPLESAMLPHP_FIRST_NAME=Clark \
-e SIMPLESAMLPHP_LAST_NAME=Kent \
kvb2univpitt/test-saml2-idp:v1
```

To check if docker is running, execute the following command:

```
docker ps -a

```
You should see output similar to this:
```
CONTAINER ID        IMAGE                            COMMAND                  CREATED             STATUS              PORTS                                                    NAMES
b0c902685caa        kvb2univpitt/test-saml2-idp:v1   "docker-php-entrypoi…"   21 seconds ago      Up 20 seconds       0.0.0.0:8080->8080/tcp, 80/tcp, 0.0.0.0:8443->8443/tcp   testsamlidp_idp
```

### Access the Identity Provider (IdP)
You can now access the SimpleSAMLphp web interface of the IdP at [http://localhost:8080/simplesaml](http://localhost:8080/simplesaml).  In this example, the admin password is ***letmein***.

### Stop the Docker
To stop docker, open up a terminal and execute the following command:

```
docker stop testsamlidp_idp
docker rm testsamlidp_idp
```

## Run the SAML 2.0 Service Provider (SP)
We are now ready to run the SAML service provider.  Download and extract the source code to a folder.  Go to the folder **saml-sp** and execute the following command to run the service provider:

```
./mvnw spring-boot:run
```

Launch your browser and go to [https://localhost:6443/login](https://localhost:6443/login).
You will see a warning for potential security risk because the SSL certificate is not officially signed.  Just click on the "Advance" button and the "Accept the Risk and Continue" button.

![SSL Warning](https://raw.githubusercontent.com/kvb2univpitt/saml-sp/main/img/ssl_warning.png)

You should see the login page.

![Welcome Page](https://raw.githubusercontent.com/kvb2univpitt/saml-sp/main/img/welcome.png)

Click the "Sign In" Button.  It should redirect you to the login page of the identity provider.  Enter the username and password.  In this example, the username is ***ckent*** and the password is ***batman***.

![Login Page](https://raw.githubusercontent.com/kvb2univpitt/saml-sp/main/img/login.png)

Once you enter the correct credentials, you should be directed back to the service provider.  You now have access to the service provider.

![Main Page](https://raw.githubusercontent.com/kvb2univpitt/saml-sp/main/img/main.png)

To shutdown the service provider, hit Ctrl-C.
