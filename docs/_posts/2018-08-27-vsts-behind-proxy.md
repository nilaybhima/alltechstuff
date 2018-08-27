---
title: "VSTS behind a Corporate Proxy"
categories:
  - VSTS
tags:
  - VSTS
last_modified_at: 2018-08-27
read_time: true
---

Recently, I had the opportunity to work for an enterprise organisation to setup a VSTS build agent on an on-prem server for achieving Continuos Integration and Continuous Delivery for their on-premise code base.

Although there as some article out there for achieving this, I thought of jotting down some important points which I had trouble with during this phase and some which were hard to look for on the web.

1. The build agent service should be started on the build server by mentioning the web proxy details. Information on this can be found here: https://docs.microsoft.com/en-us/vsts/pipelines/agents/proxy?view=vsts&tabs=windows

```yaml
./config.cmd --proxyurl http://127.0.0.1:8888 --proxyusername "1" --proxypassword "1"
```

Once you run the build agent, VSTS creates a secured tunnel via port 443 between the build server and itself. Hence, the communication is secured. 


2. When you add any task to your build/release definitions in VSTS, the code required to run this particular task is downloaded onto your build server and then executed from here. The catch here is that executables like NuGet, npm, git, etc. also require proxy details to communicate with the outside world. So, you need to mention your proxy details on every task that you add to the definitions and will require access to the web.

Some of the most common tasks that will require the proxy details are below:

* NuGet

Earlier, there had been an ongoing issue with VSTS restoring NuGet packages behind proxy servers and hence custom PowerShell tasks had to be used. This has now been resolved and you can simply add the NuGet restore task to your build definitions. In the future, if you again have trouble with this task, you can add the custom NuGet restore PowerShell Task:

```yaml
nuget config -set http_proxy=$(ProxyUrl)
nuget config -set http_proxy.user=$(ProxyUserName)
nuget config -set http_proxy.password=$(ProxyPassword)
nuget update -self\ncd $(System.DefaultWorkingDirectory)
nuget restore -ConfigFile NuGet.Config"
```

* npm

```yaml
--proxy "http://$(ProxyUserName):$(ProxyPassword)@$(ProxyUrl)/" install  -g npm@latest
```

* git

```yaml
git config --global http.proxy http://:$(ProxyPassword)@$(ProxyUrl)
```

* choco

```yaml
choco config set proxy $(ProxyUrl)
choco config set proxyUser $(ProxyUsername)
choco config set proxyPassword $(ProxyPassword)
```

* curl

```yaml
#Use the proxy flags for each curl request
curl --proxy-user "$(ProxyUserName):$(ProxyPassword)" -x $(ProxyUrl)
```