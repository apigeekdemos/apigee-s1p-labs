# Apigee Edge Service Broker Org Plan: Secure a CF App

*Duration : 20 mins*

*Persona : API Team*

# Use case

You have an Application / API Created in Pivotal Cloud Foundry. You want to secure it through a proxy in Apigee Edge.

![Org Plan](resources/OrgPlan.png)

# How can Apigee Edge help?

The [Apigee Edge Service Broker for PCF](https://docs.apigee.com/api-platform/integrations/cloud-foundry/install-and-configure-apigee-service-broker) enables developers to manage APIs for their PCF apps through the Apigee Edge management console.

This lab describes how to push a sample app to Pivotal Cloud Foundry (PCF), use an already created Apigee Edge service instance, and bind the application to it. After binding the application to the Apigee Edge service instance, requests to the app will be forwarded to an Apigee Edge API proxy for management. Its the same lab as listed in [PCF documentation](https://docs.apigee.com/api-platform/integrations/cloud-foundry/proxying-cloud-foundry-app-org-plan)


# Pre-requisites

* You have [installed and configured](http://docs.pivotal.io/partners/apigee/installing.html) the *Apigee Edge Service Broker for PCF tile*. Or you got a set of credentials from your instructor that has access to a PCF environment with *Apigee Edge Service Broker for PCF* tile. 

* You have installed [cf CLI](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html) in your laptop. Alternatively, you can use Google Cloud Shell to [install cf cli following these steps](https://gist.github.com/dzuluaga/a5f42e343dad0198d052df8b2c35a45e).

* You have an Apigee account and have access to an Apigee Org. For PCF One Environment, pick up credentials [APIGEE_LOGIN, APIGEE_USERNAME, and APIGEE_PASSWORD here](https://apigeespringone.page.link/registration).

# Instructions

**1. Register to obtain PCF Credentials:** Open [registration sheet](https://apigeespringone.page.link/registration
), pick a user and complete additional fields: name, role, email, and company.

**2. Set environment variables:** Before you begin, you will need to set the environment variables of your PCF foundation. Please ask your instructor to provision them. Replace values of **`PCF_USERNAME`** and **`PCF_PASSWORD`** variables.

   **a. Save apigee-pcf-environment.sh** 
   
   ```bash
   # apigee-pcf-environment.sh file
   #
   # e.g. apigee-pcf-user-XXX -  where XXX is your unique identifier
   export PCF_USERNAME=apigee-pcf-user-XXX
   export PCF_PASSWORD=SomePass$word
   
   # user APIGEE_USERNAME from registration sheet above
   export APIGEE_USERNAME=LOOKUP_IN_SPREADSHEET_ABOVE
   export APIGEE_PASSWORD=LOOKUP_IN_SPREADSHEET_ABOVE

   # This the the domain/hostname where the PCF is deployed. If you are using self signed certs for this endpoint, you will have to use `--skip-ssl-validation` for some of the commands
   export YOUR_SYSTEM_DOMAIN=apps.pcfone.io

   # The instance of your PCF deployment. If you are familiar with PCF, you may just refer to this as ORG. Since Apigee also as a concept of ORG, we will call this PCF_ORG for this lab and your ORG for this lab is called - "group-apigee"
   export PCF_ORG=group-apigee

   # An org can contain multiple spaces. The space you will pick for this lab is called - "apijam"
   export PCF_SPACE=apijam

   # PCF API Endpoint - e.g. - https://api.run.pcfone.io
   export PCF_API=https://api.run.pcfone.io

   # PCF Domain for your apps.  // e.g. - apps.pcfone.io
   export PCF_DOMAIN=apps.pcfone.io
   ```
   
   **b. run `source apigee-pcf-environment.sh` to see these environment variables.** 

# Steps

**1. Push the sample application as a CF app to PCF**

   a. Clone the Apigee Edge GitHub repo:
    
    $ git clone https://github.com/apigeekdemos/cloud-foundry-apigee.git

   b. Change to the *org-and-microgateway-sample* directory of the cloned repo:
    
    $ cd cloud-foundry-apigee/lab1-org-plan/

   c. In the *lab1-org-plan* directory, edit *manifest.yml* and change the **'name' parameters**:
   
   * **name**: {your_initials}-sampleapi

```yml
  applications: 
  - name: {your_initials}-sampleapi
    memory: 64M
    disk_quota: 128M 
    instances: 1 
    path: . 
    buildpack: nodejs_buildpack
```

   d. Set your API endpoint to the Cloud Controller of your deployment

```bash
$ cf api $PCF_API
Setting api endpoint to ...
OK

api endpoint:    https://api.run.pcfone.io
api version:    2.112.0
```

e. Log in to your deployment and select an org and a space

```bash
$ cf login
  -or-
$ cf login -u {PCF_USERNAME} -p {PCF_PASSWORD}


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

f. Select the org and space through the following command:

```bash
$ cf target -o $PCF_ORG -s $PCF_SPACE
```

g. Push the sample app to PCF:
    
From within the *org-and-microgateway-sample* folder run:

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
urls: as-sample.apps.pcfone.io
last uploaded: Wed Aug 29 20:29:32 UTC 2018
stack: cflinuxfs2
buildpack: nodejs_buildpack

     state     since                    cpu    memory         disk            details
#0   running   2018-08-29 01:29:55 PM   0.0%   44.7M of 64M   56.1M of 128M
```

h. Get a list of apps to determine the URL of the app just pushed:
    
```bash
$ cf apps

➜  org-and-microgateway-sample git:(master) ✗ cf apps
Getting apps in org group-apigee / space apijam as shuklaankur@google.com...
OK

name                  requested state   instances   memory   disk   urls
as-sample             started           1/1         64M      128M   as-sample.apps.pcfone.io
```

i. Use curl to send a test request to the url of the running app. Verify the response from the app. 


```bash
$ curl https://as-sample.apps.pcfone.io

{"hello":"hello from cf app"}
```

**2. Install the Apigee Broker Plugin**

a. Run the CF install-plugin command

```   
$ cf install-plugin -r CF-Community "apigee-broker-plugin"

Installing plugin Apigee-Broker-Plugin...
OK
Plugin Apigee-Broker-Plugin 0.1.1 successfully installed.
```

b. Make sure the plugin is available by running:

```
$ cf -h

...
Commands offered by installed plugins:
  apigee-bind-mg,abm     apigee-push,ap           apigee-unbind-org,auo
  apigee-bind-mgc,abc    apigee-unbind-mg,aum     
  apigee-bind-org,abo    apigee-unbind-mgc,auc  
```

**3. List your Service Instance**

Note - If you are executing this lab in your own PCF foundation, you will need to create your own Service instance, for the purpose of this lab , we have pre-created Service instance for you.

a. List the Marketplace services and locate the Apigee Edge service:

```bash
$ cf marketplace
```

```
Getting services from marketplace in org group-apigee / space apijam as shuklaankur@google.com...
OK

service                       plans                                        description
apigee-edge                   org, microgateway, microgateway-coresident   Apigee Edge API Platform
.
.
.
```
   
b. List instances of the Apigee Edge service. You will be Select the *org* service plan.

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

c. Show more details of `apigee-org-service` service instance

```
➜  org-and-microgateway-sample git:(master) ✗ cf service apigee-org-service
Showing info of service apigee-org-service in org group-apigee / space apijam as shuklaankur@google.com...

name:            apigee-org-service
service:         apigee-edge
bound apps:
tags:
plan:            org
description:     Apigee Edge API Platform
documentation:
dashboard:       https://enterprise.apigee.com/platform/#/

```

**4. Bind the Sample CF App created in Step 1 to route its requests to the Apigee Egde ORG Service Instance listed in Step 3.**

The apigee-bind-org command creates a proxy for you and binds the app to the service.

    $ cf apigee-bind-org --app {your_sample_app_name} --service apigee-org-service \
    --apigee_org amer-api-partner19 --apigee_env test --domain apps.pcfone.io  \
    --protocol https --user {apiogee_username} --pass {apigee_password}

The above command will promt for these entries. Enter the values as listed below:

Action to take ("bind", "proxy bind", or "proxy") [required]: Use `proxy bind`

The host domain to which API calls are made. Specify a value only if your Apigee proxy domain is not the same as that given by your virtual host [optional]: {press enter}

**5. Verify the binding**
    The above 'bind org' completes route binding by instructing the PCF go-router to route all traffic, headed for your application, via an Apigee ORG. You can verify that the binding was successful by using the 'cf routes' command and seeing that the 'ORG Plan' service is now associated with your application route. See below example:

```bash
$ cf routes
Getting routes for org group-apigee / space apijam as shuklaankur@google.com ...

space    host                  domain           port   path   type   apps                  service
apijam   as-sample             apps.pcfone.io                        as-sample             apigee-org-service
```

**6. Test the binding**
   
   Once you’ve bound your app’s path to the Apigee service (creating an Apigee proxy in the process), you can try it out with a simple curl command.

   This time Trace the API proxy from the Apigee Edge management console.
   * Login using the Apigee Org credentials given to you. Ensure that you are viewing the org 'amer-api-partner19' (see screenshot)

   ![Org Plan](resources/SelectOrg.png)
   * In the management console, under APIs > API proxies, locate the name of the proxy you just created with bind-route-service. It will have a name of cf-{your-initials}-sample.apps.pcfone.io
   * Click the new proxy’s name to view its Overview page.
   * Click the Trace tab, then click the Start Trace Session button.

   * From a command line run the curl command you ran earlier to make a request to your Cloud Foundry app you pushed, such as:

```bash
   $ curl http://{your_sample_app_name}.sample.apps.pcfone.io

   You should see the following response as before:

    {“hello”:“hello from cf app”}
```
    
**Congratulations!**...
    
    What does this mean
    - You have analytics across all your APIs, created through PCF
    - You can now add authentication, traffic management and other capablities to your API
    - When you do that the business teams can create API Products, and scale the consumption
    - If you have swagger spec for this API, you can enable your developers to access these APIs through smartdocs

# Unbinding the Application from 'ORG' Service Plan

Since we will be using the same application for our next set of labs, we will now be unbinding this application from the ORG plan.

To unbind the the application we will be using the 'apigee-unbind-org' comand as follows:

```bash
$ cf apigee-unbind-org --app {your_initials}-sample --domain apps.pcfone.io --service apigee-org-service

Unbinding may leave apps mapped to route as-sample.apps.pcfone.io vulnerable; e.g. if service instance apigee-org-service provides authentication. Do you want to proceed?> y
Unbinding route as-sample.apps.pcfone.io from service instance apigee-org-service in org group-apigee / space apijam as shuklaankur@google.com...
OK
➜  org-and-microgateway-sample git:(master) ✗
``` 

You can verify that the 'unbind-org' was successful by using 'cf routes' command and ensuring that you do not see the service listed against your app route:

```
$ cf routes
Getting routes for org group-apigee / space apijam as shuklaankur@google.com ...

space    host                  domain           port   path   type   apps                  service
apijam   as-sample             apps.pcfone.io                        as-sample
```

# Summary

In this lab you have added API Management to an API created in PCF using Apigee Edge.

# References

* [Installing Apigee Edge Service Broker for PCF tile]
    (http://docs.pivotal.io/partners/apigee/installing.html)
