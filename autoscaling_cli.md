---

copyright:
  years: 2019
lastupdated: "2019-04-18"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}

# Managing autoscaling through Command Line Tool
{: #autoscale_cli}

You can also use  App Autoscaler command-line interface plugin (aka AutoScaler CLI) to manage policy, query metrics and scaling history. 

## Installing AutoScaler CLI

Use the following command to install `AutoScaler CLI` which is a plugin of Cloud Foundry CLI.  

``` 
cf install-plugin -r CF-Community app-autoscaler-plugin
```
{:codeblock} 

The same command can be used to update `AutoScaler CLI` plugin to the latest version if you have a prior installation. 

## Using the AutoScaler CLI

If you already logged into a Cloud Foundry environment on {{site.data.keyword.Bluemix}} and have applications running in your space as described in [deploying apps guide][deploy_app], follow steps below to create a scaling policy for your application, and query metrics and scaling history. 

### Creating auto-scaling policy

1. (optional) Confirm the App-Autoscaler API endpoint is set correctly by default.

   If Cloud Foundry API endpoint is presented as `api.<DOMAIN>`, the App-Autoscaler API endpoint will be `autoscaler.<DOMAIN>` by default.  

   `cf autoscaling-api (aka: asa)` is used to query and modify the App-Autoscaler API endpoint setting. 
  
   ```
   cf asa
   ```
   {:codeblock} 
  
   If the App-Autoscaler API endpoint is incorrect, you need to change it with command:
  
   ```
   cf asa autoscaler.<DOMAIN>
   ```
   {:codeblock} 
   
2. Create a policy JSON file on your local machine. 

   ```
   cat > <YOUR_POLICY_FILE> << EOF
   {
          "instance_min_count": 1,
          "instance_max_count": 5,
          "scaling_rules": [
                  {
                          "metric_type": "memoryutil",
                          "breach_duration_secs": 120,
                          "threshold": 80,
                          "operator": ">=",
                          "cool_down_secs": 300,
                          "adjustment": "+1"
                  },
                  {
                          "metric_type": "memoryutil",
                          "breach_duration_secs": 120,
                          "threshold": 30,
                          "operator": "<",
                          "cool_down_secs": 300,
                          "adjustment": "-1"
                  }
          ]
   }
   EOF
   ```
   {:codeblock} 
  
   With the policy defined above, your applicaiton will be scaled based on memory utilization  when the defined threshold is breached for more than `120 seconds`.  

   Refer to [Cloud Foundry App-Autoscaler user guide][autoscaler_user_guide] to get more details for the policy specification and samples.
 
3. Attach the policy file to your application `cf attach-autoscaling-policy (aka: aasp)`

   ```
   cf aasp <APP_NAME> <PATH_TO_POLICY_FILE>
   ```
   {:codeblock} 

Then your application will be automatically scaled based on the defined policy.

Furthermore, you can use the command line interface to query metrics and scaling events.

### Query the most recent aggregated metrics

  App-Autoscaler supports multiple [metric types][metric_type], but only the metrics you defined in your policy could be retrieved, i.e. `memoryutil` in the above example.  
  
  ```
  cf asm <APP_NAME> <METRIC_TYPE> --desc
  ```
  {:codeblock} 
  
### Query scaling events

  ```
  cf ash <APP_NAME> --desc
  ```
  {:codeblock} 
  

Refer to [Cloud Foundry App Autoscaler CLI plugin guide][autoscaler_cli] for more options to use the command line. 
  

[autoscaler_project]: https://github.com/cloudfoundry/app-autoscaler
[autoscaler_user_guide]: https://github.com/cloudfoundry/app-autoscaler/blob/master/docs/Readme.md
[autoscaling_policy]: https://github.com/cloudfoundry/app-autoscaler/blob/master/docs/policy.md
[autoscaler_cli]: https://github.com/cloudfoundry/app-autoscaler-cli-plugin#cloud-foundry-cli-autoscaler-plug-in-
[metric_type]: https://github.com/cloudfoundry/app-autoscaler/blob/master/docs/Readme.md#metric-types
[deploy_app]: https://{DomainName}/docs/cloud-foundry/deploy-apps.html#dep_apps
