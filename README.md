# saml-sp
A simple Spring Boot SAML service provider template.

Requirements:
- OpenJDK 11
- Apache Maven 3.x
- Docker 19.x

## Setup SAML Identity Provider (IdP)
The first thing we need to do is to setup a SAML identity provider (IdP) to issues authentication assertions to our service provider (SP).  Download the [Docker Test SAML 2.0 Identity Provider (IdP)](https://github.com/kristophjunge/docker-test-saml-idp).  Extract the file docker-test-saml-idp-master.zip to your local directory.

### Update the Docker Software
The docker SAML IdP is built with [SimpleSAMLphp](https://simplesamlphp.org/) and PHP7 Apache [images](https://hub.docker.com/_/php/).  Let's update the SimpleSAMLphp to version **1.18.8** and PHP to version **7.4** by modifying the file ***Dockerfile*** in the **docker-test-saml-idp-master/** folder:
- Modify line 1:  Change **FROM php:7.1-apachestrong text** to **FROM php:7.4-apache**.
- Modify line 10: Change **ARG SIMPLESAMLPHP_VERSION=1.15.2** to **ARG SIMPLESAMLPHP_VERSION=**1.18.8****.

### Add Service Provider Metadata
We need to add the service provider metadata so that the identity provider can communicate with our service provider.  Go to the folder **docker-test-saml-idp-master/config/simplesamlphp/** and open up the file ***saml20-sp-remote.php***.

Replace the following:
```
$metadata[getenv('SIMPLESAMLPHP_SP_ENTITY_ID')] = array(
    'AssertionConsumerService' => getenv('SIMPLESAMLPHP_SP_ASSERTION_CONSUMER_SERVICE'),
    'SingleLogoutService' => getenv('SIMPLESAMLPHP_SP_SINGLE_LOGOUT_SERVICE'),
);
```
with the following:
```
$metadata[getenv('SIMPLESAMLPHP_SP_ENTITY_ID')] = array(
    'entityid' => getenv('SIMPLESAMLPHP_SP_ENTITY_ID'),
    'contacts' =>
    array(
    ),
    'metadata-set' => 'saml20-sp-remote',
    'expire' => 1603945030,
    'AssertionConsumerService' =>
    array(
        0 =>
        array(
            'Binding' => 'urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST',
            'Location' => getenv('SIMPLESAMLPHP_SP_ASSERTION_CONSUMER_SERVICE'),
            'index' => 1,
        ),
    ),
    'SingleLogoutService' =>
    array(
        0 =>
        array(
            'Binding' => 'urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect',
            'Location' => getenv('SIMPLESAMLPHP_SP_SINGLE_LOGOUT_SERVICE'),
        ),
    ),
    'NameIDFormat' => 'urn:oasis:names:tc:SAML:2.0:nameid-format:transient',
    'keys' =>
    array(
        0 =>
        array(
            'encryption' => false,
            'signing' => true,
            'type' => 'X509Certificate',
            'X509Certificate' => 'MIIFZTCCA02gAwIBAgIUMA0ktIryTG8BHnzg0mF7VRG6sFkwDQYJKoZIhvcNAQELBQAwQjELMAkGA1UEBhMCWFgxFTATBgNVBAcMDERlZmF1bHQgQ2l0eTEcMBoGA1UECgwTRGVmYXVsdCBDb21wYW55IEx0ZDAeFw0yMDEwMjcwNDA5NDVaFw0zMDEwMjUwNDA5NDVaMEIxCzAJBgNVBAYTAlhYMRUwEwYDVQQHDAxEZWZhdWx0IENpdHkxHDAaBgNVBAoME0RlZmF1bHQgQ29tcGFueSBMdGQwggIiMA0GCSqGSIb3DQEBAQUAA4ICDwAwggIKAoICAQDBvLXPOT5arQRtVFDOgrGFm2eff3h8z3OTCrkc26Bdf1jWy46zpc6XnoMs3p5S/oxKfR/p4AMKGaFVRkuit8SsDcVIM4B9kZUUS6u4yUDYG5a8oViqGWFKH5DlXMUa6KdSgROJeqz/di3uFXg0vNIoUExFob2B2EHwAmZ44/GIawxyNwt0m6ylJbLsVearfpAdm0A31m5Xl5ceACzZDIaCsweblypVlOGdmFnoeP0JyRjyM+wofvdfN2j+vDYFb855iAeSazX0AmfFd9FW2K1YF0yZlIcSjszqkgb6EkWJ9p3rkcR4awm3VdErXOcH7A8E3tZr5tYypTabGsYbPns6+z0+3bc2nAy9NDeLmABkK2r2+q2K+AEkWKM0DMXI+P/CaukDOBJ2mY2ZcFVG+ATQ4+VT2LrRPwzKssx88D9qbwAe4X0Tvg72xKAIwhICqybPOVURTcpGCmdpwcrYUhXzztKAIW1uMQ6Yde1NOv0YNu8+ofBvCX3/+yiZXX32QctRXoK2YcZYhMRV/d7xIOqXPmLwo/6u+o6p14krNqXku7Qser1W45+pHDV+pzRucCYrfcYR6OQKmNsTnHAqR7fMfGCzlOfFxQPJoSVhvrvvBYGKk5NfD112PRkGFGWX9NqW7ZLTWj0nO+NEZLAVVf/JTHrBOUBKhw/T7JyMZYz7hQIDAQABo1MwUTAdBgNVHQ4EFgQUdYHPIgSzzP/hjgTD0EvWsEf7SRkwHwYDVR0jBBgwFoAUdYHPIgSzzP/hjgTD0EvWsEf7SRkwDwYDVR0TAQH/BAUwAwEB/zANBgkqhkiG9w0BAQsFAAOCAgEAfv4cqBQdypwFFaG82OX4I9bco0ucKcAQ2Mrtp7HwTEI2JAc+DMd7yHnUClsZk0qYuXjYxDvVsPsWSjW+qQKDHiTwGRLtOI+Q+aokHcUY2Yr0lXP4KhAd+FFEHvVVvOu23J/TzUqFe5IEk07kkPqqdJPrIXpJJtAEyX9YM4+GeXdkEcNo8lgJFzoArheKh5FdYvy2s+/rD27al0cfbsWiKBmQ5iTe9R6G/2Yf7i2OjlNupj1JFQgebX2O3tfSo3a+Oden4CWIclZh+RMB0/R9lbrTuTYTX816WzfU/7dwdEcMXOTVHgAQhTtVjOgUkdYbnXgCl08TP8cHE4qbHhUzlTIWXgbpZFrdZhI/A4EVAe0BatuPM15l/vyKD+MOW6Ad3Y0W0/gRM8a3VG3OaMjSsExiHZF97RVlAT2uituCYfPLCtD4QlqBDocY2rlvI6sJTa8V76OM5ORY2NCgvULzLls8iTAJROLO+lbGCpti1mMCU7X0Cejlfb/WiRKFPko9EEpKiHWNuKnVCrHp8VoCoFXQn1rQO7wSdYyoz6HDNc1Sn7ix2qBbRQHgiWDXAQUhLOz9mVWlpXhzqQSpXInvUj5Tz4lu6ce+fJeP0fxIJq/CxN/SXFkDYsOLhMZpxZoDbkg13mPQr7eQqQmHMBiO3kg21sFeS+PFfCx/ELISYiA=',
        ),
        1 =>
        array(
            'encryption' => true,
            'signing' => false,
            'type' => 'X509Certificate',
            'X509Certificate' => 'MIIFZTCCA02gAwIBAgIUMA0ktIryTG8BHnzg0mF7VRG6sFkwDQYJKoZIhvcNAQELBQAwQjELMAkGA1UEBhMCWFgxFTATBgNVBAcMDERlZmF1bHQgQ2l0eTEcMBoGA1UECgwTRGVmYXVsdCBDb21wYW55IEx0ZDAeFw0yMDEwMjcwNDA5NDVaFw0zMDEwMjUwNDA5NDVaMEIxCzAJBgNVBAYTAlhYMRUwEwYDVQQHDAxEZWZhdWx0IENpdHkxHDAaBgNVBAoME0RlZmF1bHQgQ29tcGFueSBMdGQwggIiMA0GCSqGSIb3DQEBAQUAA4ICDwAwggIKAoICAQDBvLXPOT5arQRtVFDOgrGFm2eff3h8z3OTCrkc26Bdf1jWy46zpc6XnoMs3p5S/oxKfR/p4AMKGaFVRkuit8SsDcVIM4B9kZUUS6u4yUDYG5a8oViqGWFKH5DlXMUa6KdSgROJeqz/di3uFXg0vNIoUExFob2B2EHwAmZ44/GIawxyNwt0m6ylJbLsVearfpAdm0A31m5Xl5ceACzZDIaCsweblypVlOGdmFnoeP0JyRjyM+wofvdfN2j+vDYFb855iAeSazX0AmfFd9FW2K1YF0yZlIcSjszqkgb6EkWJ9p3rkcR4awm3VdErXOcH7A8E3tZr5tYypTabGsYbPns6+z0+3bc2nAy9NDeLmABkK2r2+q2K+AEkWKM0DMXI+P/CaukDOBJ2mY2ZcFVG+ATQ4+VT2LrRPwzKssx88D9qbwAe4X0Tvg72xKAIwhICqybPOVURTcpGCmdpwcrYUhXzztKAIW1uMQ6Yde1NOv0YNu8+ofBvCX3/+yiZXX32QctRXoK2YcZYhMRV/d7xIOqXPmLwo/6u+o6p14krNqXku7Qser1W45+pHDV+pzRucCYrfcYR6OQKmNsTnHAqR7fMfGCzlOfFxQPJoSVhvrvvBYGKk5NfD112PRkGFGWX9NqW7ZLTWj0nO+NEZLAVVf/JTHrBOUBKhw/T7JyMZYz7hQIDAQABo1MwUTAdBgNVHQ4EFgQUdYHPIgSzzP/hjgTD0EvWsEf7SRkwHwYDVR0jBBgwFoAUdYHPIgSzzP/hjgTD0EvWsEf7SRkwDwYDVR0TAQH/BAUwAwEB/zANBgkqhkiG9w0BAQsFAAOCAgEAfv4cqBQdypwFFaG82OX4I9bco0ucKcAQ2Mrtp7HwTEI2JAc+DMd7yHnUClsZk0qYuXjYxDvVsPsWSjW+qQKDHiTwGRLtOI+Q+aokHcUY2Yr0lXP4KhAd+FFEHvVVvOu23J/TzUqFe5IEk07kkPqqdJPrIXpJJtAEyX9YM4+GeXdkEcNo8lgJFzoArheKh5FdYvy2s+/rD27al0cfbsWiKBmQ5iTe9R6G/2Yf7i2OjlNupj1JFQgebX2O3tfSo3a+Oden4CWIclZh+RMB0/R9lbrTuTYTX816WzfU/7dwdEcMXOTVHgAQhTtVjOgUkdYbnXgCl08TP8cHE4qbHhUzlTIWXgbpZFrdZhI/A4EVAe0BatuPM15l/vyKD+MOW6Ad3Y0W0/gRM8a3VG3OaMjSsExiHZF97RVlAT2uituCYfPLCtD4QlqBDocY2rlvI6sJTa8V76OM5ORY2NCgvULzLls8iTAJROLO+lbGCpti1mMCU7X0Cejlfb/WiRKFPko9EEpKiHWNuKnVCrHp8VoCoFXQn1rQO7wSdYyoz6HDNc1Sn7ix2qBbRQHgiWDXAQUhLOz9mVWlpXhzqQSpXInvUj5Tz4lu6ce+fJeP0fxIJq/CxN/SXFkDYsOLhMZpxZoDbkg13mPQr7eQqQmHMBiO3kg21sFeS+PFfCx/ELISYiA=',
        ),
    ),
    'validate.authnrequest' => true,
    'saml20.sign.assertion' => true,
);
```

### Add Custom User
The docker IdP comes with two static users configured with the following data:

| UID | Username | Password | Group | Email |
|---|---|---|---|---|
| 1 | user1 | user1pass | group1 | user1@example.com |
| 2 | user2 | user2pass | group2 | user2@example.com |

You can add your own user by modifying the file ***authsources.php*** in the folder **docker-test-saml-idp-master/config/simplesamlphp**.   For an example, we can add a user with the following data:
| UID | Username | Password | Group | Email | Display Name | First Name | Last Name |
|---|---|---|---|---|---|---|---|
| 3 | jkim | inyourarea | blackpink | jkim@yg.com | Kim, Jennie | Jennie | Kim |

Add the following to the file ***authsources.php*** :
```
'jkim:inyouarea' => array(
    'uid' => array('3'),
    'eduPersonAffiliation' => array('blackpink'),
    'email' => 'jkim@yg.com',
    'displayName' => 'Kim, Jennie',
    'givenName' => 'Jennie',
    'sn' => 'Kim',
),
```

### Build the Docker Image
Go to the folder **docker-test-saml-idp-master** and execute the following command:
```
docker build -t local/test-saml-idp .
```
To check that the image is install, execute the command ```docker images```.
You should see output similar to the following:
```
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
local/test-saml-idp   latest              02df60f9f54d        7 seconds ago       502MB
php                   7.4-apache          aaf1fe8553ca        13 days ago         414MB
```

### Run Docker Test SAML 2.0 Identity Provider (IdP)
Open up a terminal and execute the following command:
```
docker run --name=testsamlidp_idp \
-p 8080:8080 \
-p 8443:8443 \
-e SIMPLESAMLPHP_SP_ENTITY_ID=https://localhost:6443/saml2/service-provider-metadata/samltestidp \
-e SIMPLESAMLPHP_SP_ASSERTION_CONSUMER_SERVICE=https://localhost:6443/login/saml2/sso/samltestidp \
-e SIMPLESAMLPHP_SP_SINGLE_LOGOUT_SERVICE=https://localhost:6443/login?logout \
local/test-saml-idp
```
To check if docker is running, execute the following command:
```
docker ps -a
```
You should see output like this:
```
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                                                    NAMES
f1e75aa0bc3b        local/test-saml-idp   "docker-php-entrypoi…"   8 seconds ago       Up 7 seconds        0.0.0.0:8080->8080/tcp, 80/tcp, 0.0.0.0:8443->8443/tcp   testsamlidp_idp
```

### Access the Identity Provider (IdP)
You can now access the SimpleSAMLphp web interface of the IdP at [http://localhost:8080/simplesaml](http://localhost:8080/simplesaml).  The admin password is ***secret***.

### Stop the Docker
To stop docker, open up a terminal and execute the following command:
```
docker stop testsamlidp_idp
docker rm testsamlidp_idp
```

### Delete the Docker Image
To delete the docker image, execute the following command:
```
docker rmi <IMAGE ID>
```
Note that `<IMAGE ID>` is the ID of the docker image.  In this example, it's ***02df60f9f54d***.

## Run SAML Service Provider (SP)
Download and extract the source code to a folder.  Go to the folder and execute the following command:
```
./mvnw spring-boot:run
```

Launch your browser and go to [https://localhost:6443/login](https://localhost:6443/login).

To shutdown the program, hit Ctrl-C.
