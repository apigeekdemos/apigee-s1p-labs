# Apigee Edge Service Broker Microgateway Coresident Plan: Secure a CF Springboot App

*Duration : 45 mins*

*Persona : API Team*

# Use case

![Apigee Microgateway Coresident Plan](./images/Apigee_Microgateway_Coresident.png "Apigee Microgateway Coresident Plan")

# How is ths lab different from Lab 3
Lab 3 and Lab 4 are both working on the core principle of co-resident microgateway architecture. Only difference is that the application we deploy in lab 4 is a Spring Boot app and this comes with its own web server (Apache Tomcat) and hence adds a different diamention to the deployment within PCF.

# How can Apigee Edge help?

TODO

# Pre-requisites

* You have [installed and configured](http://docs.pivotal.io/partners/apigee/installing.html) the *Apigee Edge Service Broker for PCF tile*. Or you got a set of credentials from your instructor that has access to a PCF environment with *Apigee Edge Service Broker for PCF* tile. 

* You have installed [cf CLI](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html).

* You have an Apigee account and have access to an Apigee Org.

* You have installed [Maven 3.2 or later](https://www.mkyong.com/maven/install-maven-on-mac-osx/).

* You have java 1.8 or later


TODO * EdgeMicro

# Instructions

Before you begin, you will need to get the following from your PCF instance or receive them from your instructor.

YOUR-SYSTEM-DOMAIN: This the the domian/hostname where the PCF is deployed. If you are using self signed certs for this endpoint, you will have to use `--skip-ssl-validation` for some of the commands

PCF-USER-NAME: PCF username

PCF-PASSWORD: PCF Password

PCF_ORG: The instance of your PCF deployment. If you are familiar with PCF, you may just refer to this as ORG. Since Apigee also as a concept of ORG, we will call this PCF_ORG for this lab

PCF_SPACE: An org can contain multiple spaces. This is the space you will pick for this lab

PCF_API: PCF API Endpoint

PCF_DOMAIN: PCF Domain for your apps. 

APIGEE_ORG: Apigee Edge Org Name.

APIGEE_ENV: Apigee Edge Environment.

EDGEMICRO_KEY: Edge Micro Key.

EDGEMICRO_SECRET: Edge Micro Secret.

## Step 1: Create a Service Instance (Optional)

Note - This step is optional if you have completed lab 3 as you can re-use the service created in Lab 3

### 1.a Set your API endpoint to the Cloud Controller of your deployment.

```bash
cf api --skip-ssl-validation https://api.run.pcfone.io
Setting api endpoint to api.YOUR-SYSTEM-DOMAIN...
OK
API endpoint:  https://api.YOUR-SYSTEM-DOMAIN (API version: 2.59.0)
Not logged in. Use 'cf login' to log in.
```

### 1.b Log in to your deployment and select an org and a space.

```bash
$ cf login
API endpoint: https://api.YOUR-SYSTEM-DOMAIN
Email> user@example.com
Password>
```

### 1.c List the Marketplace services and locate the Apigee Edge service:

```bash
$ cf marketplace
Getting services from marketplace in org example / space development as user@example.com...
OK

service          plans                     description
apigee-edge      org, microgateway, microgateway-coresident         Apigee Edge API Platform
```

### 1.d Create an instance of the Apigee Edge service. Select the microgateway-coresident service plan to have Apigee Edge Microgateway run in the same container as your Cloud Foundry app.

e.g. Replace INITIALS-YOUR-SERVICE-INSTANCE to something similar to dz-apigee-mg-coresident-service-instance.

```bash
$ cf create-service apigee-edge microgateway-coresident INITIALS-YOUR-SERVICE-INSTANCE -c \
'{"org":"YOUR-APIGEE-ORG", "env":"YOUR-APIGEE-ENV"}'
Creating service instance INITIALS-YOUR-SERVICE-INSTANCE in org apigee / space ...
OK
```

### 1.e Use the cf service command to display information about the service instance:

```bash
$ cf services
Getting services in org apigee / space sandeepmuru+pivotal+labuser9@google.com as sandeepmuru+pivotal+labuser9@google.com...
OK

name                service       plan                      bound apps       last operation
as-cor-mgw-apigee   apigee-edge   microgateway-coresident   as_springhello   create succeeded
```

## Step 2: Install the Plugin 

### 2.a Install the Apigee Broker Plugin as follows.

```bash
cf install-plugin -r CF-Community "apigee-broker-plugin"
Searching CF-Community for plugin apigee-broker-plugin...
Plugin apigee-broker-plugin 0.1.1 found in: CF-Community
Attention: Plugins are binaries written by potentially untrusted authors.
Install and use plugins at your own risk.
OK
Plugin Apigee-Broker-Plugin successfully uninstalled.
Installing plugin Apigee-Broker-Plugin...
OK
Plugin Apigee-Broker-Plugin 0.1.1 successfully installed.
```

### 2.b Make sure the plugin is available by running the following command:

```bash
$ cf -h
…
Commands offered by installed plugins:
  apigee-bind-mg,abm      apigee-unbind-mgc,auc    enable-diego
  apigee-bind-mgc,abc     apigee-unbind-org,auo    has-diego-enabled
  apigee-bind-org,abo     dea-apps                 migrate-apps
  apigee-push,ap          diego-apps               dev,pcfdev
  apigee-unbind-mg,aum    disable-diego
```
## Step 3: Clone Sample Java Spring Boot App

### 3.a Clone the Following GitHub repo:
```bash
$ git clone https://github.com/apigeekdemos/cloud-foundry-apigee.git
```

### 3.b Change directory to the gs-spring-boot/initial directory of the cloned repo:

```bash
$ cd gs-spring-boot/initial
```
### 3.c Create a target directory 

```bash
$ mkdir target
```
### 3.d Complete Build using Maven 

```bash
$ mvn package && java -jar target/gs-spring-boot-0.1.0.jar
```

###You should see output like this:
```bash
Lets inspect the beans provided by Spring Boot:
application
beanNameHandlerMapping
defaultServletHandlerMapping
dispatcherServlet
embeddedServletContainerCustomizerBeanPostProcessor
handlerExceptionResolver
helloController
httpRequestHandlerAdapter
messageSource
mvcContentNegotiationManager
mvcConversionService
mvcValidator
org.springframework.boot.autoconfigure.MessageSourceAutoConfiguration
org.springframework.boot.autoconfigure.PropertyPlaceholderAutoConfiguration
org.springframework.boot.autoconfigure.web.EmbeddedServletContainerAutoConfiguration
org.springframework.boot.autoconfigure.web.EmbeddedServletContainerAutoConfiguration$DispatcherServletConfiguration
org.springframework.boot.autoconfigure.web.EmbeddedServletContainerAutoConfiguration$EmbeddedTomcat
org.springframework.boot.autoconfigure.web.ServerPropertiesAutoConfiguration
org.springframework.boot.context.embedded.properties.ServerProperties
org.springframework.context.annotation.ConfigurationClassPostProcessor.enhancedConfigurationProcessor
org.springframework.context.annotation.ConfigurationClassPostProcessor.importAwareProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalRequiredAnnotationProcessor
org.springframework.web.servlet.config.annotation.DelegatingWebMvcConfiguration
propertySourcesBinder
propertySourcesPlaceholderConfigurer
requestMappingHandlerAdapter
requestMappingHandlerMapping
resourceHandlerMapping
simpleControllerHandlerAdapter
tomcatEmbeddedServletContainerFactory
viewControllerHandlerMapping
```

### 3.e Check your Java Spring Booth Application is running from another Terminal Window
```bash
shuklaankur-macbookpro:initial shuklaankur$ curl localhost:8080
Greetings from Spring Boot!shuklaankur-macbookpro:initial shuklaankur$
```
### 3.f In the 'initial' directory, open manifest.yml.

Edit manifest.yml to change the name property to "your_initials" specific to your deployment. See the following example:

```yaml
---
applications:
  - name: as_springhello
    memory: 1GB
    #buildpack: java_buildpack
    health-check-type: http
    health-check-http-endpoint: /healthcheck
    path: target/gs-spring-boot-0.1.0.jar
env:
    JAVA_OPTS: '-Dserver.port=8081'
    #APIGEE_MICROGATEWAY_PROXY: edgemicro_cf-test.local.pcfdev.io
    #APIGEE_MICROGATEWAY_CUST_PLUGINS: plugins
    #APIGEE_MICROGATEWAY_PROCESSES: 2
    APIGEE_MICROGATEWAY_CONFIG_DIR: config
    APIGEE_MICROGATEWAY_CUSTOM: |
                                 {"policies":
                                      {
                                      "oauth":
                                          {
                                          "allowNoAuthorization": false,
                                          "allowInvalidAuthorization": false,
                                          "verify_api_key_url": "https://amer-api-partner19-test.apigee.net/edgemicro-auth/verifyApiKey"
                                          }
                                      },
                                      "sequence": ["healthcheck", "oauth"]
                                      }
```

To support Cloud Foundry application health check, make sure your applications block includes the health-check-type and health-check-http-endpoint properties:

```yaml
health-check-type: http
health-check-http-endpoint: /healthcheck
```

Also, the sequence property must include a reference to the healthcheck plugin, as shown here.

```yaml
"sequence": ["healthcheck", "oauth"]
```
Note :- "healthcheck" should come before the "oauth" policy to ensure no tokens are required by Cloud Foundry to make healthcheck calls. For more on health check, see Using [Application Health Checks](https://docs.cloudfoundry.org/devguide/deploy-apps/healthchecks.html).

The following describes the manifest properties:



|Variable	| Description |
|---------|-------------|
|APIGEE_MICROGATEWAY_CONFIG_DIR	| Location of your Apigee Microgateway configuration directory. |
|APIGEE_MICROGATEWAY_CUST_PLUGINS	| Location of your Apigee Microgateway plugins directory. |
|APIGEE_MICROGATEWAY_PROCESSES |	The number of child processes that Apigee Microgateway should start. If your Microgateway performance is poor, setting this value higher might improve it. |
|APIGEE_MICROGATEWAY_CUSTOM |	“sequence” corresponds to the sequence order in the microgateway yaml file (this will be added on to the end of any current sequence in the microgateway yaml file with duplicates removed). </br>“policies” correspond to any specific configuration needed by a plugin; for instance, “oauth” has the ‘"allowNoAuthorization": true configuration. These policies will overwrite any existing policies in the microgateway yaml file and add any that do not yet exist.|


### 3.g. Save the edited file.

## Step 4: Install Apigee Edge Microgateway and Cloud Foundry App

Here, you install Apigee Edge Microgateway and your Cloud Foundry app to the same Cloud Foundry container.

### 4.a. Step not required during API Jam as these files are provided under resources folder. Execute for generating key and secrets. [Install and configure Apigee Edge Microgateway.](https://docs.apigee.com/api-platform/microgateway/2.5.x/installing-edge-microgateway.html)

## Step 5: Create config directory and copy Apigee Microgateway config file from resources

In this step we will copy microgateway config file under under Microgateway-Coresident resources folder to config folder`amer-api-partner19-test-config.yaml`. Note APIGEE_MICROGATEWAY_CONFIG_DIR reference in Manifest.yaml file to config folder. 

```bash
$ mkdir config
$ cp ~/tools/git/apijam/Labs/PivotalJam/Lab\ 3\ -\ Proxy\ a\ CF\ app\ using\ Edge\ Microgateway\ \(Microgateway-Coresident\ plan\)/resources/amer-api-partner19-test-config.yaml ./config
```

## Step 6: Check Edge Microgateway and Cloud Foundry App ports and disable oauth plugin

### 6.a Check Edge Microgateway listening on port 8080
As per [PCF requirements Applications should listen on port `8080`](https://docs.run.pivotal.io/devguide/deploy-apps/routes-domains.html#http-vs.-tcp-routes). Therefore, Edge Microgateway will frontend our Cloud Foundry app, which listens on port `8081`.

To confirm this is the case, run the following commands on the MG config file:

```bash
$ cat config/amer-api-partner19-test-config.yaml

edge_config:
edgemicro:
  port: 8080
```

**edgemicro/port is effectively listening on 8080. IMPORTANT: by default MG config file uses port 8000. So, make sure to make changes accordingly to PCF requirements.**

### 6.b. Disable oauth plugin
We will be disabling the oauth plugin on the MG config file, this is done so that the Microgatway does not manadate oauth tokens for the healthcheck calls. Oauth has already been enabled on our manifest.yaml file (see above) and the sequence defined there will ensure that all actual API traffic is subjected to Oauth policy enforcement.

To disable oauth, we need to comment out the oauth option from within the plugin sequence (see below) within the MG config file:

```bash
$ vi config/amer-api-partner19-test-config.yaml

plugins:
    sequence: 
      #- oauth
```


### 6.c. Check Spring Boot App is on Port 8081
Ensure that your Cloud Foundry app won't be running on port 8080 and instead on the port specified by the 'server.port' environment variable.

To check this open the manifest.yml file and ensure that the JAVA_OPTS emntry is present as below (under the env section):

```
$ cat manifest.yml
.
.
.
env:
      JAVA_OPTS: '-Dserver.port=8081'
.
.
.
```

This looks good. Cloud Foundry app listens on port 8081.

## Step 7: Push the Cloud Foundry App to your Cloud Foundry Container

In this step Spring Boot target application will be pushed to PCF. We will use the apigee-push command instead of regular piush command as apigee-push option automatically injects the microgateway within the cf container for the co-resident plan to work.

Note - Please enter 'y' for the "microgateway-coresident" question, other questions are required for java application). See below for Example:

```bash
shuklaankur-macbookpro:initial shuklaankur$ cf apigee-push
Do you plan on using this application with the "microgateway-coresident" plan? [y/n] y
If you are pushing a java application, enter the path to the archive. Otherwise press [Enter]: ./target/gs-spring-boot-0.1.0.jar
Path to configuration directory that contains a microgateway yaml [required]: ./config
Path to configuration directory that contains custom plugins [optional]:
Specific name of application to push [optional]:
Using manifest file /Users/shuklaankur/pcf/lab4/gs-spring-boot/initial/manifest.yml

Creating app as_springhello in org apigee / space sandeepmuru+pivotal+labuser9@google.com as sandeepmuru+pivotal+labuser9@google.com...
OK

Creating route as-springhello.apps.apigee-demo.net...
OK

Binding as-springhello.apps.apigee-demo.net to as_springhello...
OK

Uploading as_springhello...
Uploading app files from: /var/folders/mb/bdbms16n4c7c3spybl9l8t7h00hd_t/T/unzipped-app561589720
Uploading 282.4K, 86 files
Done uploading
OK
```
Note - Enter the path with file name for the .jar file. For the config file path, just point to the directory. 

## Step 7: Bind the Cloud Foundry App to the Service Instance and Restart the Target App

In this step, you bind a Cloud Foundry app to the Apigee service instance you created. The apigee-bind-mgc command creates the proxy for you and binds the app to the service. This command also gives you an option to restage your target application after binding is complete.

Each bind attempt requires authorization with Apigee Edge, with credentials passed as additional parameters to the apigee-bind-mgc command. You can pass these credentials as arguments of the apigee-bind-mgc command or by using a bearer token.

### 7.a. (Optional) If you’re using a bearer token to authenticate with Apigee Edge, get or update the token using the Apigee SSO CLI script. 

(If you’re instead using command-line arguments to authenticate with username and password, specify the credentials in the next step - 7.b.)

Another option for Cloud, although it should be your last resource if `Edge Script (approach below)`, and `acurl` fail is to [use the Apigee Management API to get a token](https://apidocs.apigee.com/api-reference/content/using-oauth2-security-apigee-edge-management-api#usingtheapitogettokens).

#### Download the Apigee Edge scripts:

```bash
$ curl https://login.apigee.com/resources/scripts/sso-cli/ssocli-bundle.zip -o "ssocli-bundle.zip"
```

#### Unzip the ssocli-bundle.zip file

This includes get_token, a script that gets or updates a token that you use to authenticate with your Apigee Edge organization. You need this token to bind the Apigee Edge route service to your app.

```bash
$ tar xvf ssocli-bundle.zip
```

#### Create a .sso-cli directory in your user directory:

$ mkdir ~/.sso-cli
Use the get_token script to create a token. When prompted, enter the Apigee Edge username and password you use to log in to your organization.

$ ./get\_token
The get_token script writes the token file into ~/.sso-cli. For more about get_token, see the Apigee documentation.

### 7.b Bind the app to the Apigee service instance with the apigee-bind-mgc command

Use the command without arguments to be prompted for argument values. To use the command with arguments, see the command reference at the end of this topic. For help on the command, type cf apigee-bind-mgc -h.

```bash
$ cf apigee-bind-mgc --app CF_APP_NAME --service CF_SERVICE_INSTANCE_NAME --apigee_org APIGEE_ORG --apigee_env APIGEE_ENV --edgemicro_key EDGEMICRO_KEY --edgemicro_secret EDGEMICRO_SECRET --target_app_route TARGET_APP_ROUTE --target_app_port 8081 --action 'proxy bind' --user APIGEE_USER_NAME --pass APIGEE_PASSWORD
```

This command should return `OK`. If it returns and error, try again until you get it. This will also give you an option to re-start your application, select 'y' to start the target application and Microgatway.

**Tip: to troubleshot `cf` commands use `-v` for verbose output.**

You’ll be prompted for the following:

|Argument |	Description |
|---------|-------------|
|Apigee username	| Apigee user name. **Provided by instructor above.** Not used if you pass a bearer token with the –bearer argument.|
|Apigee password |	Apigee password. **Provided by instructor above.** Not used if you pass a bearer token with the –bearer argument.|
|Action to take	| Required. **Creates and binds (Go router registration) proxy 'proxy bind'. Don't forget single quotes.** proxy to generate an API proxy; bind to bind the service with the proxy; proxy bind to generate the proxy and bind with a single command.|
|Apigee environment (apigee_env)	|Required. **Provided by instructor above.** The Apigee environment where your proxy should be deployed.|
|Apigee organization (apigee_org)	| Required. **Provided by instructor above.**  The Apigee organization where your proxy should be created.|
|Application to bind to (app)	|Required. **Retrieve using `cf apps` use value from name column** Name of the Cloud Foundry application to bind to.|
|Microgateway key	(edgemicro_key) | Required. **Provided by instructor.** Your Apigee Edge Microgateway key.|
|Microgateway secret (edgemicro_secret)	| Required. **Provided by instructor** Your Apigee Edge Microgateway secret.|
|Service instance name to bind to (service)	| Required. **Retrieve using `cf services`** Name of the Apigee service to bind to.|
|Target application port	| Required. **Typically 8081 as per step above after `$ cat manifest.yml` to inspect -Dserver.port=8081.** This is the Port for your Cloud Foundry app. This should not be 8080.|
|Target application route (target_app_route)	| Required. **Retrieve using `cf apps` and copy url. Do not include protocol (http or https)** The URL for your Cloud Foundry app. This will be the suffix of the proxy created for you through the bind command.|

Example:
```bash

cf apigee-bind-mgc --app as_springhello --service as-cor-mgw-apigee --apigee_org amer-api-partner19 --apigee_env test --edgemicro_key <key> --edgemicro_secret <secret> --target_app_route as-springhello.apps.apigee-demo.net --target_app_port 8081 --action 'proxy bind' --user <apigee username> --pass <apigee password>]
Binding service as-cor-mgw-apigee to app as_springhello in org apigee / space sandeepmuru+pivotal+labuser9@google.com as sandeepmuru+pivotal+labuser9@google.com...
OK
TIP: Use 'cf restage as_springhello' to ensure your env variable changes take effect
Would you like to start your application now? [y/n] y
Starting app as_springhello in org apigee / space sandeepmuru+pivotal+labuser9@google.com as sandeepmuru+pivotal+labuser9@google.com...
Downloading microgateway_decorator...
.
.
.
.
.
Destroying container
Successfully destroyed container

0 of 1 instances running, 1 starting
0 of 1 instances running, 1 starting
1 of 1 instances running

App started


OK

App as_springhello was started using this command `CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-2.0.2_RELEASE -memorySizes=metaspace:64m..,stack:228k.. -memoryWeights=heap:65,metaspace:10,native:15,stack:10 -memoryInitials=heap:100%,metaspace:100% -stackThreads=300 -totMemory=$MEMORY_LIMIT) && JAVA_OPTS="-Djava.io.tmpdir=$TMPDIR -XX:OnOutOfMemoryError=$PWD/.java-buildpack/open_jdk_jre/bin/killjava.sh $CALCULATED_MEMORY -Djava.ext.dirs=$PWD/.java-buildpack/container_security_provider:$PWD/.java-buildpack/open_jdk_jre/lib/ext -Djava.security.properties=$PWD/.java-buildpack/security_providers/java.security -Dserver.port=8081" && SERVER_PORT=$PORT eval exec $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp $PWD/. org.springframework.boot.loader.JarLauncher`

Showing health and status for app as_springhello in org apigee / space sandeepmuru+pivotal+labuser9@google.com as sandeepmuru+pivotal+labuser9@google.com...
OK

requested state: started
instances: 1/1
usage: 1G x 1 instances
urls: as-springhello.apps.apigee-demo.net
last uploaded: Mon Jun 25 22:55:16 UTC 2018
stack: cflinuxfs2
buildpack: container-security-provider=1.5.0_RELEASE java-buildpack=v3.18-offline-https://github.com/cloudfoundry/java-buildpack.git#841ecb2 java-main java-opts open-jdk-like-jre=1.8.0_131 open-jdk-like-memory-calculator=2... (with decorator microgateway-coresident)

     state     since                    cpu    memory        disk           details
#0   running   2018-06-25 04:01:07 PM   0.0%   60.1M of 1G   301.9M of 1G
shuklaankur-macbookpro:initial shuklaankur$

```

### Step 8: Verify the API Proxy has been Deployed to the environment

Verify that proxy with proxy name edgemicro_{APP_NAME.PCF_DOMAIN} create in Edge has been deployed successfuly on the corresponding environment through [https://apigee.com/edge](https://apigee.com/edge). Remember to use Apigee username and password provided by instructor.

![alt text](./images/apigee_edge_test_deployment.png "Api Proxy deployment successful")


### Step 9: Obtain API Keys by creating a Product, Developer and App

Since the microgateway has oauth policy enforced we will need an API Key in order to pass our security validation and successfully make a call to our target application. To obtain a security key we need to create a product, developer and an developer app on Apigee Edge Portal.

You can refer the notes in Lab 3 as this is the same process we followed earlier.

### Step 10: Restart the Target App and Test the App Binding

#### Step 10.a Restart App
We will need to restart out target application, so that the Edge MG is able to download the lost of products and keys from Apigee Edge. Alternatively, Edge MG will do a download every 10 mins.
```bash
 cf restart as_springhello

 Restarting app as_springhello in org apigee / space sandeepmuru+pivotal+labuser9@google.com as sandeepmuru+pivotal+labuser9@google.com...

Stopping app...

Waiting for app to start...

name:              as_springhello
requested state:   started
instances:         1/1
usage:             1G x 1 instances
routes:            as-springhello.apps.apigee-demo.net
last uploaded:     Wed 20 Jun 16:46:22 PDT 2018
stack:             cflinuxfs2
buildpack:         container-security-provider=1.5.0_RELEASE java-buildpack=v3.18-offline-https://github.com/cloudfoundry/java-buildpack.git#841ecb2 java-main java-opts open-jdk-like-jre=1.8.0_131
                   open-jdk-like-memory-calculator=2... (with decorator microgateway-coresident)
start command:     CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-2.0.2_RELEASE -memorySizes=metaspace:64m..,stack:228k..
                   -memoryWeights=heap:65,metaspace:10,native:15,stack:10 -memoryInitials=heap:100%,metaspace:100% -stackThreads=300 -totMemory=$MEMORY_LIMIT) && JAVA_OPTS="-Djava.io.tmpdir=$TMPDIR
                   -XX:OnOutOfMemoryError=$PWD/.java-buildpack/open_jdk_jre/bin/killjava.sh $CALCULATED_MEMORY
                   -Djava.ext.dirs=$PWD/.java-buildpack/container_security_provider:$PWD/.java-buildpack/open_jdk_jre/lib/ext
                   -Djava.security.properties=$PWD/.java-buildpack/security_providers/java.security -Dserver.port=8081" && SERVER_PORT=$PORT eval exec $PWD/.java-buildpack/open_jdk_jre/bin/java
                   $JAVA_OPTS -cp $PWD/. org.springframework.boot.loader.JarLauncher

     state     since                  cpu    memory         disk           details
#0   running   2018-06-25T22:46:01Z   0.0%   551.3M of 1G   301.9M of 1G
shuklaankur-macbookpro:config shuklaankur$
```

Now we are ready to test out binding with the API Key from Edge.

From command line run the curl command you ran earlier to make a request to your Cloud Foundry app (see below for example):

```bash
shuklaankur-macbookpro:config shuklaankur$ cf apps
Getting apps in org apigee / space sandeepmuru+pivotal+labuser9@google.com as sandeepmuru+pivotal+labuser9@google.com...
OK

name                requested state   instances   memory   disk   urls
as_springhello      started           1/1         1G       1G     as-springhello.apps.apigee-demo.net  
```

```bash
$ curl as-springhello.apps.apigee-demo.net -H 'x-api-key:oDQcJ4yNc9iEXifMd68cbeVh03wb5mGW'
Greetings from Spring Boot!
```

Note - If above command fails to execute or hangs in terminal, add -H 'Cache-Control: no-cache' to the curl call and it will not hang. You can also execute this with Postman client or similar tool.

The new proxy is now ready for you or someone on your team to add policies, define security, traffic management, and more. For more information refer to Edge Migrogateway Documentation on Apigee Docs.