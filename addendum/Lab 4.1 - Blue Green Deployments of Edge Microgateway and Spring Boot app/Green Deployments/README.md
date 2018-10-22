# Lab 4.1. Edge Microgateway Coresident Rollout Updates

TODO - describe purpose of blue green deployments.

#### References 
This document is inspired in the tutorial described in the following document:

https://docs.pivotal.io/pivotalcf/2-1/devguide/deploy-apps/blue-green.html

#### Create Instance Service for Green App 

```bash
cf create-service apigee-edge microgateway-coresident dz_pcf_springhello_green -c '{"org":"amer-api-partner19", "env":"test"}'
```

#### Push App

Push the second app with `Hello World App #2.`
`$ cf apigee-push --archive ./target/gs-spring-boot-0.1.0.jar --config ./config`


#### Map to existing Route
cf map-route Green2 apps.hol.apigee-pcf-apijam.com -n dz-pcf-springhello

```bash
$ cf apigee-bind-mgc --app Green --service apigee-edge-springboot-app --apigee_org amer-api-partner19 --apigee_env test --edgemicro_key 2d99ace69294c8f791404a7dd735a83b4ce08545f9f2da2cb736a7b9019989c5 --edgemicro_secret b4eb6dea5cb19ca6c784d3345c1630d3698ab2ef8041cf7843d191f3a5d5de66 --target_app_route dz-springhello.apps.hol.apigee-pcf-apijam.com --target_app_port 8081 --action 'proxy bind' --user sandeepmuru+pivotal+labuser5@google.com --pass Apigee123
```


cf bind-service Green apigee-edge-springboot-app \
    -c '{"org":"amer-api-partner19","env":"test",
      "user": "sandeepmuru+pivotal+labuser5@google.com",
      "pass": "*****",
      "action":"proxy bind",
      "target_app_route":"dz-springhello.apps.hol.apigee-pcf-apijam.com",
      "edgemicro_key":"2d99ace69294c8f791404a7dd735a83b4ce08545f9f2da2cb736a7b9019989c5",
      "edgemicro_secret":"b4eb6dea5cb19ca6c784d3345c1630d3698ab2ef8041cf7843d191f3a5d5de66",
      "target_app_port":8081}'
