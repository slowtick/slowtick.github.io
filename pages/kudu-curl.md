---
layout: default
title: Azure Kudu services
permalink: /pages/kudu-curl
---

Azure provides the excellent [Azure REST API](https://docs.microsoft.com/en-us/rest/api/) that can be used for managing all your Azure resources programatically. Azure also provides access to manage resources and services by other interfaces including [Azure CLI 2.0](https://docs.microsoft.com/en-us/cli/azure/install-az-cli2), [Azure PowerShell](https://github.com/Azure/azure-powershell), [Azure SDK for various languages](https://azure.microsoft.com/en-us/tools/) and [Kudu](https://github.com/projectkudu/kudu).

Kudu is deployed on all Microsoft Azure App Services (including Web apps, Mobile apps, Logic apps, and others) by default. Kudu provides a web based inteface, called Kudu Console, that is accessible usually by the URL `https://${sitename}.scm.azurewebsites.net`.

Kudu also has API URL end points for a wide varity of actions that can be performed on the App Service instances. These URL end points are simpler than the full fledged Azure REST API, can be invoked via simple `cURL` commands and are usable for quick &amp; dirty scripting.

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

See also

- [Kudu features](https://github.com/projectkudu/kudu/wiki)
- [App Services on Azure Portal](https://portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.Web%2Fsites)
- [Azure App Service REST API](https://docs.microsoft.com/en-us/rest/api/appservice/webapps)
