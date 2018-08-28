---
title: "Best Practices for building CI/CD Pipelines in VSTS"
categories:
  - VSTS
tags:
  - VSTS
last_modified_at: 2018-08-28
read_time: true
image:
  path: /assets/images/2018-08-27-vsts-best-practices.png
---


It is always good to follow certain principles or practices when building CI/CD pipelines in VSTS. The outcome is that your build/release pipelines remain clean, uniform, portable and re-usable.


#Branching Strategy

Follow a strict branching strategy for your source code like Git-Flow, Story-Based or Trunk-Based branching. This helps with uniformity and versioning of your build artifacts.

#Versioning of Artifacts

Follow a versioning strategy for your artifacts like Semantic Versioning. This could be done in the form of a custom script running at the end of a successful build.

#Lightweight CI Pipeline

When Continous Integration is enabled, your build definitions will run everytime a developer checks in code. Hence, the build definitions should have the least amount of tasks as possible. This avoids queing of jobs in the agent pools and accelerating Continous Delivery of your product.

#Variables and Secrets

Variables like credentials that will be used for more than one definition in VSTS should be added to Variable Groups. This helps in re-usablity of variables for your definitions. A new feature now available helps linking your variable secrets from an Azure Key Vault which can be enabled under the variable group properties.

<figure style="width: 1200px">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/images/2018_08_28_best-practices-cicd-pipelines-vsts.png" alt="">
</figure> 

#Task Groups

Task Groups help to keep certain tasks in a group so they could be used for other definitions in VSTS. One important point to note here is that the task groups should follow a certain naming convention so that you donot have trouble finding them in the long run when you have several task groups.

#Cleanup

If you are using a private build server for VSTS, it is highly recommended that you perform a clean up at the end of your build/release. This avoids choking up your server space and avoids system perfomance. You should set clean to true under Get Sources of your definition and select all build directories under clean options.

<figure style="width: 1200px">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/images/2018_08_28_best-practices-cicd-pipelines-vsts-1.png" alt="">
</figure> 

#Comments

When making changes to a definition, you should add comments when saving your changes. This helps your team to keep track of changes quicker.

#Pipeline as Code

Definitions should be treated as source code with a DevOps mindset. The definitions are defined as a YAML file and can be checked into source control instead of the visual web UI. This brings several advantages like distinct versioning of builds, source control, re-usabiliy, securing variables.

