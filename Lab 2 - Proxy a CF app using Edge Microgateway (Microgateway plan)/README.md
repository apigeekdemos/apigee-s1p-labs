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

# Instructions

Before you begin, you will need to get the following from your PCF instance or receive them from your instructor.

YOUR-SYSTEM-DOMAIN: This the the domian/hostname where the PCF is deployed. If you are using self signed certs for this endpoint, you will have to use `--skip-ssl-validation` for some of the commands

PCF_USERNAME: PCF Username   // e.g. - apigee-pcf-user-XXX -  where XXX is your unique identifier

PCF_PASSWORD: PCF Password

PCF_ORG: The instance of your PCF deployment. If you are familiar with PCF, you may just refer to this as ORG. Since Apigee also as a concept of ORG, we will call this PCF_ORG for this lab and your ORG for this lab is called - "group-apigee"

PCF_SPACE: An org can contain multiple spaces. The space you will pick for this lab is called - "apijam"

PCF_API: PCF API Endpoint   // e.g. - https://api.run.pcfone.io

PCF_DOMAIN: PCF Domain for your apps.  // e.g. - apps.pcfone.io

# Steps

**1. (Optional) Push the sample target application as a CF app to PCF**

<b>Note</b> - This step is optional if you have completed Lab 1, as we will be using the same sample application as Lab 1 and this application is already present in our CF Environment<br>
   a. Clone the Apigee Edge GitHub repo:
    
    $ git clone https://github.com/apigeekdemos/cloud-foundry-apigee.git

   b. Change to the *org-and-microgateway-sample* directory of the cloned repo:
    
    $ cd cloud-foundry-apigee/samples/org-and-microgateway-sample

   c. In the *org-and-microgateway-sample* directory, edit *manifest.yml* and change the 'name' parameter:
   
    name: {your_initials}-sampleapi
```
  applications: 
  - name: {your_initials}-sampleapi
    memory: 64M
    disk_quota: 128M 
    instances: 1 
    path: . 
    buildpack: nodejs_buildpack
```
d. Set your API endpoint to the Cloud Controller of your deployment
```
    $ cf api https://api.run.pcfone.io
Setting api endpoint to ...
OK

api endpoint:    https://api.run.pcfone.io
api version:    2.112.0
```
e. Log in to your deployment and select an org and a space

    $ cf login
    -or-
    $ cf login -u {PCF_USERNAME} -p {PCF_PASSWORD}
```
➜  apigee-s1p-labs git:(master) cf login
API endpoint: https://api.run.pcfone.io

Email> shuklaankur@google.com

Password>
Authenticating...
OK

Targeted org group-apigee

Targeted space apijam



API endpoint:   https://api.run.pcfone.io (API version: 2.112.0)
User:           shuklaankur@google.com
Org:            group-apigee
Space:          apijam
```

f. Select the org and space through the following command
```
$cf target -o $PCF_ORG -s $PCF_SPACE
```

g. Push the sample app to PCF:
    
From within the *org-and-microgateway-sample* folder run:
```bash
$ cf push
```
    
If successful, you should see some output from this command and finally:
```
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
urls: as-sample.apps.pcfone.io
last uploaded: Wed Aug 29 20:29:32 UTC 2018
stack: cflinuxfs2
buildpack: nodejs_buildpack

     state     since                    cpu    memory         disk            details
#0   running   2018-08-29 01:29:55 PM   0.0%   44.7M of 64M   56.1M of 128M
```

h. Get a list of apps to determine the URL of the app just pushed:
    
$ cf apps
```
➜  org-and-microgateway-sample git:(master) ✗ cf apps
Getting apps in org group-apigee / space apijam as shuklaankur@google.com...
OK

name                  requested state   instances   memory   disk   urls
as-sample             started           1/1         64M      128M   as-sample.apps.pcfone.io

```

i. Use curl to send a test request to the url of the running app. Verify the response from the app. 
    
    $ curl as-sample.apps.pcfone.io
```
{"hello":"hello from cf app"}
```

**2. List your Service Instance**

Note - If you are executing this lab in your own PCF foundation you will need to create your own Service instance, for the purpose of this lab we have pre-created Service instance for you.
   a. List the Marketplace services and locate the Apigee Edge service:
    
    $ cf marketplace
```
Getting services from marketplace in org group-apigee / space apijam as shuklaankur@google.com...
OK

service                       plans                                        description
apigee-edge                   org, microgateway, microgateway-coresident   Apigee Edge API Platform
.
.
.
```
   b. List instances of the Apigee Edge service. You will be Selecting the *microgateway* service plan for the purpose of this lab.
```
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
```
org-and-microgateway-sample git:(master) ✗ cf service apigee-microgateway-service
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

   a. Clone the Apigee Microgateway repository.
    
    $ git clone https://github.com/apigee-internal/microgateway.git
    
    $ cd microgateway
    
    $ git checkout tags/v.2.5.4
    
   b. Copy the Microgateway configuration YAML file *amer-api-partner19-test-config.yaml* from this Lab 2 */resources* folder to the *microgateway/config* directory in the Microgateway repository cloned in step c. above.
```
   $ cp resources/amer-api-partner19-test-config.yaml microgateway/config
```
   c. Edit the application manifest file *microgateway/manifest.yml* in the cloned Edge Microgateway repository to update the following env values: 

      i) Replace {your-initials} with your own for name parameter. 
      ii) Add the EDGEMICRO_KEY & EDGEMICRO_SECRET - see below 
      iii) Change the EDGEMICRO_ENV and EDGEMICRO_ORG - see below
      
      Leave the other values as-is.
```
applications:
- name: {your-initials}-edgemicro-app
  memory: 128M
  disk_quota: 512M
  instances: 1
  path: .
  buildpack: nodejs_buildpack
  env: 
    EDGEMICRO_KEY: '6f10ad9a16f85a14fa8c8595cf7cf39b7c3432bb8135a95af56ffe0776454eaf'
    EDGEMICRO_SECRET: '94102b540cc60aed1a6737f476c3cbcef5bf2d8a1a2a3273dc32e24cda990d05'
    EDGEMICRO_CONFIG_DIR: './config'
    EDGEMICRO_ENV: 'test'
    EDGEMICRO_ORG: 'amer-api-partner19'
    NODE_TLS_REJECT_UNAUTHORIZED: '0'
```
   d. Now your are ready push the Edge Microgateway as its own cloud foundy app to PCF. Run cf push from within the microgateway folder of the cloned repository.
    
    $ cf push
```
...
0 of 1 instances running, 1 starting
1 of 1 instances running

App started


OK

App as-edgemicro-app was started using this command `npm start`

Showing health and status for app as-edgemicro-app in org group-apigee / space apijam as shuklaankur@google.com...
OK

requested state: started
instances: 1/1
usage: 256M x 1 instances
urls: as-edgemicro-app.apps.pcfone.io
last uploaded: Wed Aug 29 23:29:10 UTC 2018
stack: cflinuxfs2
buildpack: nodejs_buildpack

     state     since                    cpu    memory          disk             details
#0   running   2018-08-29 04:30:26 PM   0.0%   52.9M of 256M   331.3M of 512M
```

**4. Bind the Sample CF App created in Step 1 / Lab 1 (ORG Plan - remember we are using the same target application) to route its requests to the Apigee Egde Service Instance listed in Step 2.**

   The apigee-bind-mg command creates a proxy for you and binds the app to the service.

    $ cf apigee-bind-mg --app {your_sample_target_app_name} --service apigee-microgateway-service --apigee_org amer-api-partner19 --apigee_env test --micro {your_edgemicro_app_name}.apps.pcfone.io --domain apps.pcfone.io --protocol https --user {username} --pass {password}

   The above command will promt for these entries. Enter the values as listed below:

   Action to take ("bind", "proxy bind", or "proxy") [required]: proxy bind
   Target application protocol [optional]: https


**6. Test the binding**
   
   Once you’ve bound your app’s path to the Apigee service (creating an Apigee proxy in the process), you can try it out with the sample app.

   From a command line run the curl command you ran earlier to make a request to your Cloud Foundry app you pushed, such as:
```
   $ curl http://{your_sample_app_name}.apps.pcfone.io

{"error":"missing_authorization","error_description":"Missing Authorization header"}
```
    You should see an validation error as edge micro is checking for security! 
	
**7. Test the binding again**
   
   In order to fix the error from the previous step, you need an API key.
	
   a. To get an API Key, go to Management UI, create an API Product add `edgemicro-auth` and `edgemicro_cf-{your-initials}-sampleapi-mg.YOUR-SYSTEM-DOMAIN` API Proxies to it. Create an APP and get a Key. 
	
   b. Come back to the CF CLI to restart the edge micro app, for it to get the latest API Products.

	$ cf apps
	
    $ cf restart {your_initials}-edgemicro-app

   c. Resend the request to your app this time passing the apikey as a request header.
    
    $ curl http://{URL OF YOUR APP} -H "x-api-key: {api-key}"
    
    NOTE: If curl hangs on this command, use the Postman client to make the request.
```
    {“hello”:“hello from cf app”}
```

**8. Extra credit**
    
    Login to [https://apigee.com/edge](https://apigee.com/edge)
    
    Go to API Proxies. You should see an API Proxy created by the PCF Service Broker- with the following name `edgemicro_cf-{your_initials}_helloapi.YOUR-SYSTEM-DOMAIN`
	
	You will also see `edgemicro-auth` API Proxy. Where requests are sent to for authentication. As edge microgateway does validation, you can see the validation calls coming to this API Proxy
    
    Select the API and select `TRACE` tab on the top right
    Click on the `Start Trace Session`, the green button on the top left
	Send a request to the same endpoint, as you did in step 2 
	
	$ curl https://{URL OF YOUR APP}"
      
    If you forgot the URL OF YOUR APP, you can get if through the following command (the output will have a urls section corresponding to your app)

    $ cf apps
    
**Congratulations!**...
    
    What does this mean
    - You have analytics across all your APIs, created through PCF
    - You can add authentication, traffic management and few more directly from your cf CLI, without logging into Apigee
    - When you do that the business teams can create API Products, and scale the consumption
    - If you have swagger spec for this API, you can enable your developers to access these APIs through smartdocs

# Summary

In this lab you have added API Management to an API created in PCF using Apigee Edge Microgateway.

# References

* [Installing Apigee Edge Service Broker for PCF tile]
    (http://docs.pivotal.io/partners/apigee/installing.html)


