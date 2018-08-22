---
title: "Setup Jekyll on Windows"
excerpt: "How to setup Jekyll on Windows"
tags: 
    - Jekyll
    - CodePage
date:   2018-08-05 00:00:00 -0600
#categories: powershell
---

Jekyll is a great tool that you can use to generate personal blogs or company website using markdown language. 

Here is some awesome instructions on [Run Jekyll on Windows](http://jekyll-windows.juthilo.com/){:target="_blank"} 

And even better, there is an out of box template that you can use to [create a blog with "Minimal Mistakes" theme](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/){:target="_blank"}

If you are still unsure, just clone my repo [here](https://github.com/bwwilliam/minimal-mistakes){:target="_blank"} and run the command below using your favourite CLI, such as Powershell, Bash, Cmd etc.

```bash
jekyll serve --watch
```

You may come across the error below, that is most likely due to the **Code Page** settings.
```diff
-  Conversion error: Jekyll::Converters::Scss encountered an error while converting 'assets/css/main.scss':
-                    Invalid GBK character "\xE2" on line 54
```
eg. **Powershell**<br/>![My helpful screenshot]({{"/assets/images/setup-jekyll-on-windows/codepagesetttings.JPG"}})


```bash
Set-ExecutionPolicy RemoteSigned
New-Item -Path $Profile -ItemType file -Force
```

Then, copy/paste "`chcp 65001 >$null`" into the file and save, the "*Current Code Page*" should become "`65001 (UTF-8)`"
Run the command below again
```bash
jekyll serve --watch
```
