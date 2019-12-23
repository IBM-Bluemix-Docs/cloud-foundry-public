---

copyright:
  years: 2019
lastupdated: "2019-12-23"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}

# Autoscaling
{: #autoscale_cloud_foundry_apps}

{{site.data.keyword.Bluemix}} has a built-in autoscaling support for Cloud Foundry applications to adjust application instance number automatically through 
  * Dynamic scaling based on application performance metrics.
  * Scheduled scaling based on time.

This capability is offered based on Cloud Foundry open source project [App-Autoscaler][autoscaler_project]. Refer to [user guide][autoscaler_user_guide] to get started. 

## Managing autoscaling from the console

If you already have an application deployed on Cloud Foundry,  access the autoscaling page with the following steps:

1. From the [{{site.data.keyword.cloud_notm}} console ![External link icon](../icons/launch-glyph.svg "External link icon")](https://{DomainName}){: new_window}, click the **Menu** icon ![Menu icon](../icons/icon_hamburger.svg), and select **Resource List**.
2. On the **Resource List** page, click **Cloud Foundry Apps**.
3. Click the application to view its **Overview** page.
4. Select the **Autoscaling** in the left navigation pane.
5. In the **Policy** tab configure your scaling policy for the application.

If you don't have an Cloud Foundry application running on {{site.data.keyword.Bluemix}}, [deploy a Cloud Foundry application](/docs/cloud-foundry-public?topic=cloud-foundry-public-deployingapps) to {{site.data.keyword.Bluemix}} first. 

## Managing autoscaling from the CLI

You can use  App Autoscaler command-line interface plug-in (aka AutoScaler CLI) to manage policy, query metrics and scaling history. 

### Installing AutoScaler CLI
Use the following command to install `AutoScaler CLI` which is a plugin of Cloud Foundry CLI.  

``` 
cf install-plugin -r CF-Community app-autoscaler-plugin
```
{:codeblock} 

The same command can be used to update `AutoScaler CLI` plugin to the latest version if you have a prior installation. 

### Using the AutoScaler CLI

If you already logged in to a Cloud Foundry environment on {{site.data.keyword.Bluemix}} and have applications running in your space as described in [deploying apps guide][deploy_app], follow steps below to create a scaling policy for your application, and query metrics and scaling history. 

 *  (optional) Confirm the App-Autoscaler API endpoint is set correctly by default.  

    If Cloud Foundry API endpoint is presented as `api.<DOMAIN>`, the App-Autoscaler API endpoint should be `autoscaler.<DOMAIN>`.  
    Use the command below to check the App-Autoscaler API endpoint setting.

    ```
    cf asa
    ```
    {:codeblock} 

    If the App-Autoscaler API endpoint is incorrect, you need to reset it with command:

    ```
    cf asa autoscaler.<DOMAIN>
    ```
    {:codeblock} 


*  Create a policy JSON file on your local machine. 

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
                            "cool_down_secs": 120,
                            "adjustment": "+1"
                    },
                    {
                            "metric_type": "memoryutil",
                            "breach_duration_secs": 120,
                            "threshold": 10,
                            "operator": "<",
                            "cool_down_secs": 120,
                            "adjustment": "-1"
                    }
            ]
    }
    EOF
    ```
    {:codeblock} 

    The policy above is used to trigger  scaling based on memory utilization  when the defined threshold is breached for at least `120    seconds`.  Refer to [Cloud Foundry App-Autoscaler user guide][autoscaler_user_guide] for how to create your own autoscaling policy.

*  Attach the policy to your application

    ```
    cf aasp <YOUR_APP> <YOUR_POLICY_FILE>
    ```
    {:codeblock} 

*  (optional) Furthermore, you can query the most recent aggregated metrics of your application. App-Autoscaler supports multiple [metric types][metric_type], but only the metrics you defined in your policy could be retrieved, i.e. `memoryutil` in above example.  

    ```
    cf asm <YOUR_APP> <METRIC_TYPE> --desc
    ```
    {:codeblock} 

*  (optional) Use the command below to query scaling history:

    ```
    cf ash <YOUR_APP> --desc
    ```
    {:codeblock} 

    Refer to [Cloud Foundry App Autoscaler CLI plugin guide][autoscaler_cli] for more options to use the command line. 
    
## Policy configuration

Autoscaling determines when and how to scale your application's capacity according to the autoscaling policy that you defined.  You can create the autoscaling policy from scratch or import it from an existing JSON file.  

To create a policy from scratch, follow steps below:

1. Click  **Create Auto-Scaling policy**
2. Set **Default Instance Limits** of your application with **Minimum Instance Count** and **Maximum Instance Count** to restrict the scope of resource consumption 
3. Create a series of **Dynamic Scaling Rules** with selected metric types, thresholds and adjustment strategies.  The following describes related fields in the dynamic scaling rule.
      
   **Metric types:** The following metric types are supported in dyanmic scaling rule:
   * **Memory used** represents the absolute value of the used memory of your application. The unit of _memory utilization_ unit is "MB".
   * **Memory utilization**, a short name of `memory utilization`, is the used memory of the total memory allocated to the application in percentage. For example, if the memory usage of the application is 100MB  and memory quota is 200MB, the value of _memory utilization_ is 50%.   
   * **CPU** , a short name of `cpu utilization`, is the cpu percentage used by the application. The unit of **cpu** is "%".  
      _Note_: `cpu utilization` may be affected by the total workload of the hosting hardware and other factors.
   * **Response time** represents the average amount of time the application takes to respond to a request in a given time period.  The unit of _Response time_ is "ms" (milliseconds).
   * **Throughput** is the total number of the processed requests  in a given time period. The unit of _throughput_ is "rps" (requests per second).
   * **Custom metric** 
     You can define your own metric name using a combination of alphabets and numbers, and then emit the coresponding metrics to App Autoscaler in order to trigger dynamic scaling. See details in [Custom metric usage guide][custom_metric_usage_guide].

   **Operator**, **Threshold**, **Breach Duration** , **Adjustment** and **Cooldown**: 

   Once the **threshold** is continuously breached during the **breach duration** period,  App Autoscaler will trigger a scaling action to ** adjust** application instance number unless **Cool down** setting is not fulfilled. 
   
   * **Operator** could be defined as `>=`, `>`, `<=` or `<`.
   * **Threshold** must be defined as a numeric value.  
   * **Breach duration** is defined in `seconds`.  Optional, default value is 120 seconds.
   * **Adjustment** defines how to change the number of application instances in each scaling action.  You can specify an absolute number or a percentage of insatnces to add or remove.
   * **Cooldown** defines the time duration to wait before the next scaling action takes place. A cooldown period helps to ensure that your application does not launch new instances or terminate old instances before your application becomes stable. **Cooldown** is defined in `seconds`. **Cooldown**  is an optional setting as well, and the default value is 300 seconds. 
    
4. (Optional) Define **Schedules** to scale your application during a set of time periods.

   You can define _Schedules_ to prepare enough instances for peak hours. The schedule policy overwrites the default instance limits and set the instance number to **Initial Minimum Instance Count** to enforce the instance number at the beginning of a schedule. 

   To define **Schedules**:

   * Select a **Time Zone** in which a schedule will take place.
   * Define either **Recurring schedules**  which execute periodically, or **Specific schedules** that take effect only once during specific time periods. 

   _Note_: During these scheduled time periods, all dynamic scaling rules are still effective. 

You can also import a policy file in JSON through **Import From JSON**.  Refer to [policy specification][autoscaler_user_guide] for the details of JSON format of autosacling policy. This function is particularly helpful when you [migrate from the legacy Auto-scaling service][migrate_guide].

## Metric statistics

Once a policy is defined with dynamic scaling rules, you will be able to view the latest metric values, and  metrics history in in the past 2 hours. 

_Note_: the metric value is not the raw data from each application instance but averaged over all the application instances. 
    
## Scaling history

The **Scaling history** tab allows you to query the scaling events within the past 30 days. It also shows the error messages if the scaling attempt failed. 


## Related readings

* [Best practices when setting up autoscaling policies](https://www.ibm.com/cloud/blog/autoscale-your-cloud-foundry-applications-on-ibm-cloud)
* [Bring Your Own Metrics to Autoscale Your IBM Cloud Foundry Applications](https://www.ibm.com/cloud/blog/bring-your-own-metrics-to-autoscale-your-ibm-cloud-foundry-applications)




[autoscaler_project]: https://github.com/cloudfoundry/app-autoscaler
[autoscaler_user_guide]: https://github.com/cloudfoundry/app-autoscaler/blob/master/docs/Readme.md
[autoscaling_policy]:https://github.com/cloudfoundry/app-autoscaler/blob/master/docs/policy.md
[autoscaler_cli]: https://github.com/cloudfoundry/app-autoscaler-cli-plugin#cloud-foundry-cli-autoscaler-plug-in-
[autoscaling_cfee_doc]: https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-autoscale_cloud_foundry_apps#autoscale_cloud_foundry_apps
[metric_type]: https://github.com/cloudfoundry/app-autoscaler/blob/master/docs/Readme.md#metric-types
[deploy_app]: https://{DomainName}/docs/cloud-foundry/deploy-apps.html#dep_apps
[legacy-autoscaling-catalog]: https://{DomainName}/catalog/services/auto-scaling
[legacy-autoscaling-cli]: https://{DomainName}/docs/cli?topic=auto-scaling-cli-autoscalingcli#bx_as_policy_show
[migrate_guide]: https://{DomainName}/docs/cloud-foundry-public?topic=cloud-foundry-public-autoscale_migration
[autoscaler_cli]: https://{DomainName}/docs/cloud-foundry-public?topic=cloud-foundry-public-autoscale_cli
[custom_metric_usage_guide]: https://github.com/cloudfoundry/app-autoscaler/tree/develop/docs#auto-scale-your-application-with-custom-metrics

