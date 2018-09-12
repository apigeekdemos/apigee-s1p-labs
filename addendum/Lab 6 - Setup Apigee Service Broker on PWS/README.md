# Setting Up Apigee Service Broker as a Cloud Foundry App within Pivotal Web Services (PWS)

## Background

Note:  For Pivotal Cloud Foundry users, this broker is also packaged as a *tile* for easy  installation by an Operator. See the Apigee docs for more on [installing and configuring the tile](http://docs.apigee.com/api-services/content/install-and-configure-apigee-service-broker).

This lab will focus on how you can deploy an Apigee Service Broker within an PWS enviroment using the CF CLI.

## What is <b>PWS</b> and Why use it?
<i>Pivotal Web Services</i> is an application hosting platform that lets your agile developers focus on applications not infrastructure, manage multiple environments, and deploy production apps reliably. By signing up for a free PWS account online you can trial Pivotal cloud foundry without any need for infrastructre investments in a cloud foundary foundation up front.


## Apigee Service Broker
Apigee Service Broker manages connections between your Cloud Foundry app and Apigee Edge (on premise or cloud) or Microgateway running in the same Cloud Foundry environment. Typically the Apigee Servoice broker is installed via an Pivotal Operations Manager tile and is available to all ORG's within that foundation. However, since we are using an PWS account for our labs we do not have access to Operations Manager (OpsMan), we will be deploying the Service broker as an cloud foundry application and then using this application to instantiate the <i>space scoped service broker</i> and eventually creating our service offerings.  


1. [Ensure you have the prerequisites.](apigee-service-broker-prerequisites.md)
1. [Install the service broker](#install) to make it available in the marketplace (CF administrator/operator).
2. [Create an instance of the service broker](#instance) for your Cloud Foundry org/space (CF user).
3. [Create Service Offerings](#service) a service to an app as needed (CF user).

> Steps in the instructions use CF CLI with an Apigee plugin. To see corresponding CF CLI commands, see [Mapping for Apigee and Cloud Foundry integration commands](mapping-apigee-cf-cli.md). 

## <a name="install"></a>Step 1: Install the Apigee service broker from source

Now that you have signed up for your own PWS account, you're the Cloud Foundry ORG administrator for your ORG and you can now install a service broker as an application (in other words, a broker app). This is particularly useful when running a Cloud Foundry development environment.


These instructions assume a [PWS trial account](https://console.run.pivotal.io/) environment, at the domain `cfapps.io`. If you're using another kind of Cloud Foundry host, be sure to adjust URLs accordingly.

1. Login - From a command prompt, log in to the Cloud Foundry instance where you'll be installing the Apigee service broker.

    ```bash
    $ cf login -a <your.endpoint> -u <username> -o <organization> -s <space>
    ```
Example: 
    ```bash
    $ ➜  apijam git:(master) cf login -a https://api.run.pivotal.io -u shuklaankur@google.com -o ankur-org -s development
API endpoint: https://api.run.pivotal.io

Password>
Authenticating...
OK

Targeted org ankur-org

Targeted space development



API endpoint:   https://api.run.pivotal.io (API version: 2.116.0)
User:           shuklaankur@google.com
Org:            ankur-org
Space:          development
➜  apijam git:(master)
    ```


1. Clone this github project to get the service broker source you'll need.

    ```bash
    $ cd <your working directory>
    $ git clone https://github.com/ankurshukla80/cloud-foundry-apigee.git
    $ cd cloud-foundry-apigee/apigee-cf-service-broker
    ```
2. Load dependencies and test (requires that Node.js is installed).

    ```bash
    $ npm install
    $ npm test
    ```

3. In the apigee-cf-service-broker directory, edit the manifest.yml file to make the following changes:

    3 a) Change the <i>name</i> property for your application and add your initials at the end to make this parameter unique:
    
    ```bash
    ---
        applications:
        - name: apigee-cf-service-broker-as
           memory: 25M
           command: node server.js
    ```
    3 b) (Optional) change the required variables (``org`` and ``env``) and override defaults as appropriate for your environment and Apigee Edge account. (We have pre-populated this yaml file with Apigee Demo Org that you can use for the purpose of this lab) 

    Item | Purpose | Default (for SaaS Edge)
    ---- | ---- | ----
    APIGEE_DASHBOARD_URL | URL for Apigee Edge management UI | `https://enterprise.apigee.com/platform/#/`
    APIGEE_MGMT_API_URL | The endpoint URL to the Apigee Edge management API. The Apigee Edge Service Broker uses this URL when making requests to create new Apigee Edge API proxies for managing requests to PCF apps. | `https://api.enterprise.apigee.com/v1`
    APIGEE_PROXY_DOMAIN | The domain name that Cloud Foundry apps use when making calls to your API proxy. This is the domain that clients use to make calls to your APIs. Change this value if your proxy domain is not the default domain for Apigee public cloud. For example, you might have your own API domain -- such as one created with a custom virtual host. Enter the domain name here. | `apigee.net`
    APIGEE_PROXY_HOST_TEMPLATE | ES6 template literal for generated proxy host. The template that describes how the Apigee Edge host name should be generated. <br/> This represents the hostname that clients use to make calls to your APIs. Change this value if your hostname is not created in the default way -- from your Apigee org an environment names. For example, if your APIs use a custom virtual host, you might have just a domain name:<br/>`${domain}`<br />Cloud Foundry apps use this host when making calls to your API proxy. The template generates the host name from values specified when binding the CF app to the service. (Note that without any placeholders, this will be used as a literal value.) | `${org}-${env}.${domain}`
    APIGEE_PROXY_NAME_TEMPLATE | ES6 template literal for generated proxy | `cf-${route}`
    ORG | The Apigee Edge organization with proxies that will handle calls to your app. |
    ENV | The Apigee Edge environment with proxies that will handle calls to your app. |

   > **Note:** Cloud Foundry does not allow ``${...}`` syntax to be present in the environment variable section. So when changing `APIGEE_PROXY_HOST_TEMPLATE` or `APIGEE_PROXY_NAME_TEMPLATE`, be sure to not use ``${...}`` in your changes.

    ```yaml
      env:
        APIGEE_CONFIGURATIONS: |
                            [{“org”:”your-apigee-org1”,
                            “env”:”your-apigee-env1”,
                            “apigee_proxy_domain”:”apigee.net”,...},
                            <repeat the preceding for multiple orgs and envs>]
    ```

4. Deploy the Apigee service broker from the source in this repository.

    ```bash
    $ cf push
    ...
    requested state: started
    instances: 1/1
    usage: 25M x 1 instances
    urls: apigee-cf-service-broker-as.cfapps.io
    last uploaded: Thu Aug 9 18:46:38 UTC 2018
    stack: cflinuxfs2
    buildpack: nodejs_buildpack

        state     since                    cpu    memory         disk           details
        #0   running   2018-08-09 11:48:41 AM   0.0%   20.6M of 25M   168.4M of 1G
    ```
    Make a note of the broker app's URL, which you'll use to create the service broker later. Here's an example:

    ```bash
    urls: apigee-cf-service-broker-as.cfapps.io
    ```

5. Choose a user name and password and store them as environment variables for the broker app. Then restage the broker app to load those variables.

    Communication with the broker is protected with a user name and password (to prevent unauthorized access to the broker app from other sources). These credentials are specified when the broker is created, and then used for each call. However, validating those credentials is the responsibility of the broker app, which does not have those credentials provided by the runtime.

    ```bash
    $ cf set-env apigee-cf-service-broker-as SECURITY_USER_NAME <pick a username>
    $ cf set-env apigee-cf-service-broker-as SECURITY_USER_PASSWORD <pick a password>
    $ cf restage apigee-cf-service-broker-as
    ```

6. Use the credentials you just established, along with the URL for the broker app, to create the <i>space scoped</i> service broker in Cloud Foundry.

    ```bash
    $ cf create-service-broker apigee-edge <security-user-name> <security-user-password> https://apigee-cf-service-broker-as.cfapps.io --space-scoped
    ```

7. Confirm that the service broker is showing up in your Cloud Foundry marketplace and is showing the 3 available service plan offerings.

    ```bash
    $ cf marketplace
    $ cf marketplace -s apigee-edge
    Getting service plan information for service apigee-edge as shuklaankur@google.com...
    OK

    service plan              description                                                                                                                       free or paid
    org                       Apigee Edge for Route Services                                                                                                    free
    microgateway              Apigee Edge Microgateway for Route Services. This plan requires launching Microgateway as a separate Cloud Foundry application    free
    microgateway-coresident   Apigee Edge Microgateway coresident plan. Coresident means Microgateway will be on the same container as the target application   free
    apigee-cf-service-broker git:(master) ✗
    ```
## <a name="instance"></a>Step 2: Install the plugin

1. Install the Apigee Broker Plugin as follows.

    ```
    $ cf install-plugin -r CF-Community "apigee-broker-plugin"
    Searching CF-Community for plugin apigee-broker-plugin...
    Plugin apigee-broker-plugin 0.1.1 found in: CF-Community
    Attention: Plugins are binaries written by potentially untrusted authors.
    Install and use plugins at your own risk.
    Do you want to install the plugin apigee-broker-plugin? [yN]: y
    Starting download of plugin binary from repository CF-Community...
    7.85 MiB / 7.85 MiB [===========================================================================================================================================================================================================================================] 100.00% 11s
    Installing plugin Apigee-Broker-Plugin...
    OK
    Plugin Apigee-Broker-Plugin 0.1.1 successfully installed.
    ```

1. Make sure the plugin is available by running the following command:

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


## Uninstalling the service instance and broker

To uninstall the service instance, use the delete-service command.
```bash
$ cf delete-service myapigee
$ cf delete-service-broker apigee-edge
```