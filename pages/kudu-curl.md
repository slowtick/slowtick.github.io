---
layout: default
title: Kudu
permalink: /pages/kudu-curl
---

Kudu can execute a wide variety of actions via URLs

{% highlight shell %}
### Zip API

# upload content to site folder in Web App instance
curl -X PUT --data-binary @content.zip https://${sitename}.scm.azurewebsites.net/api/zip/site/ -u "user:pass"

# download files to a zip from a Web App instance folder
curl -X GET -o site-backup.zip https://${sitename}.scm.azurewebsites.net/api/zip/site/ -u "user:pass"


### VFS API

# list files
curl -X GET https://${sitename}.scm.azurewebsites.net/api/vfs/site/ -u "user:pass"

# download a file
curl -X GET https://${sitename}.scm.azurewebsites.net/api/vfs/site/${customlogsfolder}/${myapp.log} -u "user:pass"


### Site Extensions API

# get list of extensions installed
curl -X GET https://${sitename}.scm.azurewebsites.net/api/siteExtensions -u "user:pass"

# get list of all extensions available for install from https://www.siteextensions.net/ Gallery
curl -X GET https://${sitename}.scm.azurewebsites.net/api/extensionfeed -u "user:pass"

# install a MS extension from https://www.siteextensions.net/packages
curl -X PUT https://${sitename}.scm.azurewebsites.net/api/siteExtensions/Microsoft.ApplicationInsights.AzureWebSites -d "" -u "user:pass"

# install Python 2.7.11 x64 from Gallery
curl -X PUT https://${sitename}.scm.azurewebsites.net/api/siteExtensions/python2711x64 -d "" -u "user:pass"


### WebJobs API

# list webjobs
curl -X GET https://${sitename}.scm.azurewebsites.net/api/webjobs -u "user:pass"

{% endhighlight %}
