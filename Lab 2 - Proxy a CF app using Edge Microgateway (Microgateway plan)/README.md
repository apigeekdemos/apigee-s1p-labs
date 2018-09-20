# Apigee Edge Service Broker Microgateway Plan: Secure a CF App

*Duration : 20 mins*

*Persona : API Team*

# Use case

You have an API Created in Pivotal Cloud Founday. You want to proxy it through Apigee Edge using Edge Microgateway.

![Microgateway Plan](resources/MicrogatewayPlan.png)

# How can Apigee Edge help?

The [Apigee Edge Service Broker for PCF](https://docs.apigee.com/api-platform/integrations/cloud-foundry/install-and-configure-apigee-service-broker) enables developers to manage APIs for their PCF apps through the Apigee Edge management console.

This lab describes how to push a sample app to Pivotal Cloud Foundry (PCF), create an Apigee Edge service instance using Edge Microgateway, and bind the application to it. After binding the application to the Apigee Edge service instance, requests to the app will be forwarded to an Apigee Edge API proxy running on Edge Microgateway for management. Its the same lab as listed in [PCF documentation](https://docs.apigee.com/api-platform/integrations/cloud-foundry/proxying-cloud-foundry-app-microgateway-plan)

In the process described here, the PCF app and Edge Microgateway app are in separate Cloud Foundry containers.

# Pre-requisites

* You have [installed and configured](http://docs.pivotal.io/partners/apigee/installing.html) the *Apigee Edge Service Broker for PCF tile*. Or you got a set of credentials from your instructor that has access to a PCF environment with *Apigee Edge Service Broker for PCF* tile. 

* You have installed [cf CLI](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html).

* You have an Apigee account and have access to an Apigee Org.

# Instructions (Optional, if you have done Lab - 1 Org Plan)

[Lab 1 Instructions](https://github.com/apigeekdemos/apigee-s1p-labs/tree/master/Lab%201%20-%20Proxy%20a%20CF%20app%20using%20Edge%20(Org%20plan)#instructions)

# Steps

**1. Push the sample target application as a CF app to PCF**

<b>Note</b> - This step (a - Clone Github) is optional if you have completed Lab 1, as we will be using the same code Repo we cloned in Lab 1. <br>
   **a. Clone the Apigee Edge GitHub repo:**
    
    $ git clone https://github.com/apigeekdemos/cloud-foundry-apigee.git

   **b. Change to the *lab2-microgateway-plan* directory of the cloned repo:**
    
    $ cd cloud-foundry-apigee/lab2-microgateway-plan/

   **c. In the *lab2-microgateway-plan* directory, edit `manifest.yml` and change the 'name' parameter:**

```
name: {your-username}-samplebackend-lab2
```

```bash
applications:
- name: {your-username}-samplebackend-lab2
  memory: 64M
  disk_quota: 128M
  instances: 1
  path: .
  buildpack: nodejs_buildpack
```
   **d. Push the sample app to PCF:**
    
From within the *lab2-microgateway-plan* folder run:

```bash
$ cf push
```
    
If successful, you should see some output from this command and finally:

```bash
.
.
1 of 1 instances running

App started


OK

App as-sample was started using this command `npm start`

Showing health and status for app as-sample in org group-apigee / space apijam as shuklaankur@google.com...
OK

requested state: started
instances: 1/1
usage: 64M x 1 instances
urls: as-sample-mg.apps.pcfone.io
last uploaded: Wed Aug 29 20:29:32 UTC 2018
stack: cflinuxfs2
buildpack: nodejs_buildpack

     state     since                    cpu    memory         disk            details
#0   running   2018-08-29 01:29:55 PM   0.0%   44.7M of 64M   56.1M of 128M
```

**e. Get a list of apps to determine the URL of the app just pushed:**

```bash
$ cf apps

Getting apps in org group-apigee / space apijam as shuklaankur@google.com...
OK

name                  requested state   instances   memory   disk   urls
{your_app_name}       started           1/1         64M      128M   {your_app_name}.apps.pcfone.io

```

**f. Use curl to send a test request to the url of the running app. Verify the response from the app.**

```bash
$ curl https://{your_app_name}.apps.pcfone.io

{"hello":"hello from cf app"}
```

**2. List your Service Instance**

Note - If you are executing this lab in your own PCF foundation you will need to create your own Service instance, for the purpose of this lab we have pre-created Service instance for you.
   
**a. List the Marketplace services and locate the Apigee Edge service:**

```bash
$ cf marketplace

Getting services from marketplace in org group-apigee / space apijam as shuklaankur@google.com...
OK

service                       plans                                        description
apigee-edge                   org, microgateway, microgateway-coresident   Apigee Edge API Platform
.
.
.
```

**b. List instances of the Apigee Edge service. You will be Selecting the *microgateway* service plan for the purpose of this lab.**

```bash
$ cf services

Getting services in org group-apigee / space apijam as shuklaankur@google.com...
OK

name                          service       plan                      bound apps   last operation
apigee-coresident-service     apigee-edge   microgateway-coresident                create succeeded
apigee-microgateway-service   apigee-edge   microgateway                           create succeeded
apigee-org-service            apigee-edge   org                                    create succeeded
...
```

List out details of the 'microgateway' plan service instance using below command:

```bash
$ cf service apigee-microgateway-service

Showing info of service apigee-microgateway-service in org group-apigee / space apijam as shuklaankur@google.com...

name:            apigee-microgateway-service
service:         apigee-edge
bound apps:
tags:
plan:            microgateway
description:     Apigee Edge API Platform
documentation:
dashboard:       https://enterprise.apigee.com/platform/#/
....
```

**3. Deploy Edge Microgateway onto Pivotal Cloud Foundry**

**a. Clone the Apigee Microgateway repository.**
```bash    
$ git clone https://github.com/apigeekdemos/microgateway.git

$ cd microgateway

```    

**b. View to ensure that file apigee-pec-test-config.yaml exists within folder lab2-microgateway-plan/microgateway/config**
This file is contains the config details required by edge Migrogateway to communicate with Edge Enterprise on Cloud.

```bash
$ view config/apigee-pec-test-config.yaml
```
**c. Notice Edge Microgateway configuration**

```yaml
  plugins:
    sequence:
      - cloud-foundry-route-service
      - cors
      - oauth
      - spikearrest
cors:
   cors-origin: '*'
spikearrest:
   timeUnit: minute
   allow: 10
```

**d. Edit the manifest.yml file and replace {your-username} with your PCF Username (see below):**

Replace `name: {your-username}-edgemicro-app-lab2` variable with your own username. e.g. `johndoe-edgemicro-app-lab2`.

The file should look like this:
      
```yaml
applications:
- name: {your-username}-edgemicro-app-lab2
  memory: 512M
  disk_quota: 512M
  instances: 1
  path: .
  buildpack: nodejs_buildpack
  env: 
    # Replace with EDGEMICRO_KEY from Instructions above
    EDGEMICRO_KEY: 6f70f4c24077058b45464606b788a854f3631a8c57ca3cc0ed144917a33f0e1a
    # Replace with EDGEMICRO_SECRET from Instructions above
    EDGEMICRO_SECRET: 24173cbee9a078f5d2947dbcf5998724e2139e195e8f9efbcb3b3447869d4a0d
    EDGEMICRO_CONFIG_DIR: './config'
    # Replace EDGEMICRO_ORG with APIGEE_ORG variable from Instructions above
    EDGEMICRO_ORG: 'apigee-pec'
    # Replace EDGEMICRO_ENV with APIGEE_ENV variable from Instructions above
    EDGEMICRO_ENV: 'test'
    NODE_TLS_REJECT_UNAUTHORIZED: 0
```

**e. Now your are ready push the Edge Microgateway as its own cloud foundy app to PCF. Run cf push from within the microgateway folder of the cloned repository.**
```   
$ cf push

...
0 of 1 instances running, 1 starting
1 of 1 instances running

App started


OK

App as-edgemicro-app-lab2 was started using this command `npm start`

Showing health and status for app as-edgemicro-app-lab2 in org group-apigee / space apijam as xxx...
OK

requested state: started
instances: 1/1
usage: 256M x 1 instances
urls: as-edgemicro-app-lab2.apps.pcfone.io
last uploaded: Wed Aug 29 23:29:10 UTC 2018
stack: cflinuxfs2
buildpack: nodejs_buildpack

     state     since                    cpu    memory          disk             details
#0   running   2018-08-29 04:30:26 PM   0.0%   52.9M of 256M   331.3M of 512M
```

**4. Bind the Sample CF App we pushed in Step 1(d), above, to route its requests to the Apigee Egde Microgateway Service Instance listed in Step 2.b**

   The bind-route-service CF command creates a proxy for you within Apigee Edge and also binds the CF app to the service.

```bash
cf bind-route-service $PCF_DOMAIN $PCF_MGW_SERVICE_INSTANCE --hostname {your-username}-samplebackend-lab2 \
-c '{"org":"'$APIGEE_ORG'","env":"'$APIGEE_ENV'",
  "user": "'$APIGEE_USERNAME'","pass":"'$APIGEE_PASSWORD'",
  "micro": "{your-username}-edgemicro-app-lab2.apps.pcfone.io",
  "action": "proxy bind",
  "protocol":"https"}'
  
Binding route {your_app_name}-samplebackend-lab2.apps.pcfone.io to service instance apigee-microgateway-service in org group-apigee / space apijam as xxx...
OK
```


**5. Test the binding**
   
   Once you’ve bound your app’s path to the Apigee service (creating an Apigee proxy in the process), you can try it out with the sample app.

   From a command line run the curl command you ran earlier to make a request to your Cloud Foundry app you pushed, such as:

```bash
$ curl https://{your_user_num}-samplebackend-lab2.apps.pcfone.io
{"message":"no match found for /{your_app_name}-samplebackend-lab2.apps.pcfone.io","status":404}

You should see this error as edge micro does not contain the latest proxy we created via the service binding! 
```
In order to fix the error from the previous step, you need to restart your microgateway application.
	
**a. Come back to the CF CLI to restart the edge micro app, for it to get the latest API Proxies.**

```bash
$ cf apps

$ cf restart {your-username}-edgemicro-app-lab2
```

**b. Resend the request to your app this time passing the apikey as a request header.**

```bash
$ curl https://{your_user_num}-samplebackend-lab2.apps.pcfone.io

{"hello":"hello from cf app"} 
```

**6. Extra credit**
    
Login to [https://apigee.com/edge](https://apigee.com/edge)
    
Go to API Proxies. You should see an API Proxy created by the PCF Service Broker- with the following name `edgemicro_cf-{your-username}_helloapi.YOUR-SYSTEM-DOMAIN`
	
You will also see `edgemicro-auth` API Proxy. Where requests are sent to for authentication. As edge microgateway does validation, you can see the validation calls coming to this API Proxy
    
Select the API and select `TRACE` tab on the top right
Click on the `Start Trace Session`, the green button on the top left
Send a request to the same endpoint, as you did in step 2 
	
```bash
$ curl https://{URL OF YOUR APP}"
```   
If you forgot the URL OF YOUR APP, you can get if through the following command (the output will have a urls section corresponding to your app)

```bash
$ cf apps
```
    
## **Congratulations!**...
    
What does this mean
- You have analytics across all your APIs, created through PCF
- You can add authentication, traffic management and few more directly from your cf CLI, without logging into Apigee
- When you do that the business teams can create API Products, and scale the consumption
- If you have swagger spec for this API, you can enable your developers to access these APIs through smartdocs

# Summary

In this lab you have added API Management to an API created in PCF using Apigee Edge Microgateway.

# References

* [Installing Apigee Edge Service Broker for PCF tile](http://docs.pivotal.io/partners/apigee/installing.html)


