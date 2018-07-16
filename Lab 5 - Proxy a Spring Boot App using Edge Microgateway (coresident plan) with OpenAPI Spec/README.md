# Proxy a Spring Boot App using Edge Microgateway (coresident plan) enabled OpenAPI Spec

*Duration : 20 mins after running Lab #4*

*Persona : API Team*

# Use case

![Apigee Microgateway Coresident Plan](./images/Apigee_Microgateway_Coresident.png "Apigee Microgateway Coresident Plan")

# How is ths lab different from Lab 4
Lab 4 is both working on the core principle of co-resident microgateway architecture. The only difference is that the Spring Boot app classes have been annotated to generate OpenAPI (FKA as Swagger) Specification using SpringFox. These annotations enable developers to leverage additional tooling available from products like Apigee with a suite of utilities to design, document, and test. The framework used in this example

# Step 1: Run Spring Boot App from your local machine
This tutorial comes with a Spring Boot precompiled ready for deployment to a PCF foundation. Generated from [Documenting Spring Boot REST API with Swagger and SpringFox from Vojtech Ruzicka](https://www.vojtechruzicka.com/documenting-spring-boot-rest-api-swagger-springfox/).

```bash
java -jar target/springfox-example-1.0.0-SNAPSHOT.jar
```

# Step 2: Test your OpenAPI docs

**Paths**
All OpenAPI Resources(groups) http://localhost:8080/v2/api-docs
OpenAPI UI endpoint: http://localhost:8080/swagger-ui.html


# Step 3: Follow all steps from Lab #4

All the steps in this labs are the same except that jar file to be deployed to PCF is different and it contains classes with annotations from SpringFox framework.

In summary, steps from Lab #4 will:

1. Create a Service Instance
2. Push the app with the jar file
3. Bound pushed app to the Service Instance

# Step 4: Test links above with PCF Apps URLs
In my case my app will have same resources as the ones available from my local app, except that my Spring Boot instrumented with OpenAPI spec now is proxied by Edge Microgateway.


# Step 5: Test your App on PCF with Edge Microgateway

**Paths**

All OpenAPI Resources(groups) 

```bash
$ curl https://dz-springfox-example-app.apps.hol.apigee-pcf-apijam.com/v2/api-docs -H 'x-api-key:yty2x3mP5A1otbSwmiKRAY4ipMOonupF' -H 'Cache-Control: no-cache' -k
```

Retrieve all persons:

```bash
curl https://dz-springfox-example-app.apps.hol.apigee-pcf-apijam.com/v2/persons/ -H 'x-api-key:yty2x3mP5A1otbSwmiKRAY4ipMOonupF' -H 'Cache-Control: no-cache' -k
```


Note: Testing through your browser will require providing `x-api-key` and `Cache-Control` headers. Using a plugin such as [ModHeaders](https://chrome.google.com/webstore/detail/modheader/idgpnmonknjnojddfkpgkljpfnnfcklj/related?hl=en) can set those headers.

Congrats if you made it to this point!

# Step 6: Create OpenAPI Spec and copy from the terminal

In this step we'll use the spec above to generate an API Portal in Apigee. Please copy and paste the spec in raw format and create a new spec directly from Apigee.

![alt text][./images/copy-openapi-spec-to-edge.png]


# References

The integration is handled by SpringFox, a 3rd party library for enabling Swagger on Spring MVC projects.
For more information on SpringFox, please visit http://springfox.io

This tutorial is based on the article from Braeldung [Setting Up Swagger 2 with a Spring REST API](http://www.baeldung.com/swagger-2-documentation-for-spring-rest-api). 

# Support

Please report any issues to apigee-pec@google.com.