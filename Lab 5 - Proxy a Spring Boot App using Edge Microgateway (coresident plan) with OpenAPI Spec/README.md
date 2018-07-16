# Proxy a Spring Boot App using Edge Microgateway (coresident plan) enabled OpenAPI Spec

*Duration : 20 mins after running Lab #4*

*Persona : API Team*

# Use case

![Apigee Microgateway Coresident Plan](./images/Apigee_Microgateway_Coresident.png "Apigee Microgateway Coresident Plan")

# How is ths lab different from Lab 4
Lab 5 works on the core principle of co-resident microgateway architecture. The main difference with Lab 4 is that classes in the Spring Boot app have been annotated with Spring http://springfox.io to identify resources, verbs, headers, query parameters, payload, etc. So, that when Spring Boot starts OpenAPI (FKA as Swagger) Specs are generated available for use through a API endpoint.

# Step 1: Run Spring Boot App from your local machine
This tutorial comes with a Spring Boot precompiled ready for deployment to a PCF foundation. Generated from the git repo `https://github.com/dzuluaga/springfox-example`.

To run the Spring Boot app:
```bash
java -jar target/springfox-example-1.0.0-SNAPSHOT.jar
```

## Optional step to compile source code:

The jar above can be generated and run by executing this command: 
```bash
$ git clone https://github.com/dzuluaga/springfox-example.git
```
```bash
$ mvn package && java -jar target/springfox-example-1.0.0-SNAPSHOT.jar
```

Note [PersonController](https://github.com/dzuluaga/springfox-example/blob/master/src/main/java/com/vojtechruzicka/springfoxexample/controllers/PersonController.java) class annotations. These annotations are parsed during runtime and OpenAPI Spec is generated without manual intervention.

```java
@RestController
@RequestMapping("/v2/persons/")
@Api(description = "Set of endpoints for Creating, Retrieving, Updating and Deleting of Persons.")
public class PersonController {

    private PersonService personService;

    @RequestMapping(method = RequestMethod.GET, produces = "application/json")
    @ApiOperation("${personcontroller.getallpersons}")
    public List<Person> getAllPersons(@RequestHeader("x-api-key") String apiKey, @RequestHeader(value = "Cache-Control", defaultValue="no-cache") String cacheControl) {
        return personService.getAllPersons();
    }

    @RequestMapping(method = RequestMethod.GET, path = "/{id}", produces = "application/json")
    @ApiOperation("${personcontroller.getpersonbyid}")
    public Person getPersonById(@ApiParam(value = "Id of the person to be obtained. Cannot be empty.", required = true)
                                    @PathVariable int id,
                                    @RequestHeader("x-api-key") String apiKey, @RequestHeader(value = "Cache-Control", defaultValue="no-cache") String cacheControl) {
        return personService.getPersonById(id);
    }

    @RequestMapping(method = RequestMethod.DELETE, path = "/{id}")
    @ApiOperation("${personcontroller.deleteperson}")
    public void deletePerson(@ApiParam(value = "Id of the person to be deleted. Cannot be empty.", required = true)
                                 @PathVariable int id,
                                 @RequestHeader("x-api-key") String apiKey, @RequestHeader(value = "Cache-Control", defaultValue="no-cache") String cacheControl) {
        personService.deletePerson(id);
    }

    @RequestMapping(method = RequestMethod.POST, produces = "application/json")
    @ApiOperation("${personcontroller.createperson}")
    public Person createPerson(@ApiParam("Person information for a new person to be created.")
                                   @RequestBody Person person,
                                   @RequestHeader("x-api-key") String apiKey, @RequestHeader(value = "Cache-Control", defaultValue="no-cache") String cacheControl) {
        return personService.createPerson(person);
    }


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

![alt text](./images/copy-openapi-spec-to-edge.png "Logo Title Text 1")


# References

The integration is handled by SpringFox, a 3rd party library for enabling Swagger on Spring MVC projects.
For more information on SpringFox, please visit http://springfox.io

This tutorial is based on the article [Documenting Spring Boot REST API with Swagger and SpringFox from Vojtech Ruzicka](https://www.vojtechruzicka.com/documenting-spring-boot-rest-api-swagger-springfox/). 

# Support

Please report any issues to apigee-pec@google.com.