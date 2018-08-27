---
title: "Passing an array of Strings in the override template parameters in VSTS"
categories:
  - VSTS
  - Azure
tags:
  - VSTS
  - Azure
  - ARM Templates
last_modified_at: 2018-08-27T14:25:52-05:00
read_time: true
---

Recently, I wanted to deploy an Azure App Service along with different slots from a VSTS Release Definition.

For this, I had to override an Array parameter that accepted slot names in the ARM template using the Azure Resource Group Deployment Task.

<figure style="width: 1200px">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/images/2018_08_27_vsts-azure-array-override-param_override_template_parameters.png" alt="">
</figure> 

Array parameters and complex objects were not supported by VSTS until last August 2017. VSTS by default fails the build/release if you pass the array string naturally that is accepted by PowerShell Modules usually. I tried all sorts of combinations to get this working like:

```yaml
-slots @("foo","bar","foobar")
-slots ["foo", "bar", "foobar"]
-slots "["foo", "bar", "foobar"]"
```

The following syntax turned out to be the correct syntax for passing an array of strings to override a parameter in VSTS.

```yaml
-slots ["foo","bar","foobar"]
```

