---

copyright:
  years: 2015, 2020
lastupdated: "2020-08-14"

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

# Configuration options
{: #configuration_options}
{: shortdesc}

A variety of options are available for configuring the
sdk-for-nodejs buildpack.

## NPM scripts
{: #npm_scripts}
NPM provides a scripting facility allowing you to run scripts, including **preinstall** and **postinstall** scripts which are applied before and after your node_modules are installed.  See [npm-scripts](https://docs.npmjs.com/misc/scripts){: external} for complete details.

## Cache behavior
{: #cache_behavior}
{{site.data.keyword.cloud}} maintains a cache directory per node application, that is persisted between builds. The cache stores resolved dependencies so they are not downloaded and installed every time the app is deployed.  For example, suppose myapp depends on **express**.  Then the first time myapp is deployed the **express** module is downloaded.  On subsequent deployments of myapp,  the cached instance of **express** is used. The default behavior is to cache all node_modules installed by NPM and bower_components installed by bower.

Use the NODE_MODULES_CACHE variable to determine whether or not the Node buildpack uses or ignores the cache from previous builds. The default value is true.  To disable caching set NODE_MODULES_CACHE to false, for example via the {{site.data.keyword.cloud_notm}} command line:
```
    ibmcloud cf set-env myapp NODE_MODULES_CACHE false
```
{: pre}

Note that node_modules that are included in your application are not cached.

You can use a **cacheDirectories** array in your top-level **package.json** to achieve fine grained control over what modules are cached.  When the **cacheDirectories** element is present in the **package.json** only those modules which are in the **cacheDirectories** array will be cached.  In the following example only node_modules and bower_components are cached.
```
{
  "cacheDirectories": ["node_modules","bower_components"],
  ...
}
```
{: codeblock}


