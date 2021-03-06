---

copyright:
  years: 2015, 2020
lastupdated: "2020-09-09"

keywords: cloud foundry

subcollection: cloud-foundry-public



---


{:beta: .beta}
{:codeblock: .codeblock}
{:deprecated: .deprecated}
{:download: .download}
{:external: target="_blank" .external}
{:faq: data-hd-content-type='faq'}
{:gif: data-image-type='gif'}
{:help: data-hd-content-type='help'}
{:important: .important}
{:java: data-hd-programlang="java"}
{:javascript: data-hd-programlang="javascript"}
{:new_window: target="_blank"}
{:note: .note}
{:pre: .pre}
{:preview: .preview}
{:screen: .screen}
{:shortdesc: .shortdesc}
{:support: data-reuse='support'}
{:table: .aria-labeledby="caption"}
{:tip: .tip}
{:troubleshoot: data-hd-content-type='troubleshoot'}
{:tsCauses: .tsCauses}
{:tsResolve: .tsResolve}
{:tsSymptoms: .tsSymptoms}

# Integrate third-party services using hooks
{: #hooks}

You can use hooks to easily integrate third-party services in the SDK for Node.js buildpack. Note that {{site.data.keyword.IBM}} does not provide support for any third-party services that you integrate. For more information about support, see [Third-party services](/docs/cloud-foundry-public?topic=cloud-foundry-public-buildpack_support_statement).

The SDK for Node.js buildpack includes the Dynatrace hook. Dynatrace enables app monitoring of Node.js apps. Learn more about using the Dynatrace hook in the buildpack in the [Dynatrace documentation](https://www.dynatrace.com/support/help/technology-support/cloud-platforms/cloud-foundry/other-deployments-and-configurations/deploy-oneagent-on-pivotal-web-services-for-application-only-monitoring/){: external}.


When following instructions in third-party documentation, remember to use the `ibmcloud cf` command instead of `cf` to run commands for your {{site.data.keyword.cloud_notm}} buildpack.
{: tip}


