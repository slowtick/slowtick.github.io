---
layout: default
title: Timeout settings for Azure Application Gateway and App Services 
description: Various timeout settings available in Azure App Service and Azure Application Gateway
permalink: /pages/az-appgw-timeouts
jsonld_include: az-appgw-timeouts.json
index_list_at_position: 3
index_list_link_text: Timeout settings available in Microsoft Azure App Services and Microsoft Azure Application Gateway
---

## Timeout settings for Azure Application Gateway and App Services

Findings about various timeout settings available in Azure Application Gateway and Azure App Services

### `requestTimeout` in Azure Application Gateway

Azure Application Gateway is a load balancer and web application firewall (WAF) in Azure, used for load distrubution, SSL termination, prevention against web based attacks (like Cross-site scripting, SQL Injection, etc) and its other features. Typically the Azure Application Gateway would be configured to route the requests to backend App Service instances to service the request.

The Application Gateway provides settings to timeout / terminate incoming requests if the backend App Service instance takes longer to process request. Following [Azure CLI 2.0](https://docs.microsoft.com/en-us/cli/azure/install-az-cli2) based commands can be used to list the timeouts configured in the Application Gateway.

{% highlight shell %}
# Find the Resource Group that has the Application Gateway/App Service
#  (adding --output table to below would show in tabular format)
az group list --query '[].name'

# Find the Application Gateway name
#  (replace MY-RES-GROUP your Azure Resource Group value)
az network application-gateway list --resource-group MY-RES-GROUP --query '[].name'

# Get the request timeout (requestTimeout) configured in the Application Gateway
#  (replace MY-RES-GROUP and MY-APP-GATEWAY to your values)
az network application-gateway show --resource-group MY-RES-GROUP --name MY-APP-GATEWAY --query 'backendHttpSettingsCollection[].{name: name, requestTimeout: requestTimeout}'

{% endhighlight %}

### Timeouts in Azure App Service (applicationHost)

Azure App Services (including Mobile apps, Web apps, Logic apps, and others) typically run latest version of Internet Information Services (IIS 10) tweaked for Azure. Some of the basic settings for this IIS instance could be configured via [Azure Portal - App Service blade](https://portal.azure.com/). Timeout settings in Application Host level (`<system.applicationHost>`) can be configured via below means.

## `connectionTimeout` in `system.applicationHost/webLimits` of `applicationHost.config`

The [`webLimits`](https://www.iis.net/configreference/system.applicationhost/weblimits#005) element in `applicationHost.config` specifies a default connection time out of 2 minutes. This setting could not be changed directly in the `applicationHost.config` file as Azure App Service does not allow to edit the file, instead supports a mechanism called XML Document Transform (XDT) which allows to append/update values to the default `applicationHost.config` files.

The App Service merges the default `applicationHost.config` and the user defined file `applicationHost.xdt` and uses the merged file for its configuration. Following XML content can set the `connectionTimeout of webLimits` to 5 minutes. Note - this XDT content need to be uploaded to the file path `D:\home\site\applicationHost.xdt` on the App Service instance.

{% highlight xml %}
<?xml version="1.0"?>
<configuration xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">
  <system.applicationHost>
    <webLimits xdt:Transform="SetAttributes(connectionTimeout)" connectionTimeout="00:05:00">
  </system.applicationHost>
</configuration>
{% endhighlight %}

## `connectionTimeout` in `system.applicationHost/sites/site` of `applicationHost.config`

The [`limits`](https://www.iis.net/configreference/system.applicationhost/sites/site/limits#005) element in `applicationHost.config` applies site wide connection timeout and defaults to 2 minutes. This can be updated using the XDT merging mechanism. Following is XDT directive for setting `connectionTimeout` of site limits to 6 minutes and is to be kept in the file `D:\home\site\applicationHost.xdt`. Note - if there is an existing applicationConfig.xdt file, the element `sites` and its child elements from below XDT content can be added to the `<system.applicationHost>` in existing applicationConfig.xdt file.

{% highlight xml %}
<?xml version="1.0"?>
<configuration xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">
  <system.applicationHost>
    <sites>
      <site name="%XDT_SITENAME%" xdt:Locator="Match(name)">
        <limits xdt:Transform="SetAttributes(connectionTimeout)" connectionTimeout="00:06:00" />
      </site>
    </sites>
  </system.applicationHost>
</configuration>
{% endhighlight %}

### Timeouts in Azure App Service (webServer)

Using IIS's Delegating Configuration feature, App Service lets user to override some of the IIS settings. This can be done via custom config file, named web.config, placed in root folder of the site's default application (typically this path will be `D:\home\site\wwwroot\web.config`). This user defined web.config supports configuring [`<system.webServer>`](https://www.iis.net/configreference/system.webserver) and many of its child elements.

Following Azure CLI 2.0 and `cURL` calls to Azure Kudu VFS API commands can be used to get currently configured values in web.config

{% highlight shell %}
# get .scm. hostname for the app service
#  (replace MY-RES-GROUP your Azure Resource Group value)
az appservice web list --resource-group MY-RES-GROUP --query "[].{appServiceName: name, scmUrl: enabledHostNames[?contains(@, '.scm.') == \`true\`]}"
{% endhighlight %}

The default path to application web.config would be `https://{myapp}.scm.azurewebsites.net/api/vfs/site/wwwroot/web.config`. In case if you have multiple virtual path or changed default physical path, the following command can be used to get all paths that can have application level web.config file in the App Service instance.

{% highlight shell %}
# get physical path on app service instance to web.config
az appservice web config show --resource-group MY-RES-GROUP --name MY-APPSERVICE-APP --query 'virtualApplications[].{virtualPath: virtualPath, physicalPath: physicalPath}'
{% endhighlight %}

Following command can be used to get the contents of web.config

{% highlight shell %}
# replace the SCM_URL with your App Service instance SCM URL 
curl -X GET https://${SCM_URL}/api/vfs/site/wwwroot/web.config -u "user:pass"
{% endhighlight %}

## `request`, `activity` and `idle` timeout for `fastCgi` based Python and PHP apps in Azure App Service

`fastCgi` directive in `<system.webServer>` configures the FastCGI module for executing requests using Python or PHP runtime and is used for running Python applications like Django, Flask, etc as well as PHP applications.

[`requestTimeout`, `activityTimeout` and `idleTimeout` attributes of `<fastCgi>/<application>`](https://www.iis.net/configreference/system.webserver/fastcgi/application#005) can be used to control timeouts for the Python or PHP application.

{% highlight xml %}
<?xml version="1.0"?>
<configuration>
  <appSettings>
    <add key="WSGI_ALT_VIRTUALENV_HANDLER" value="easterngrocery_web.app"/>
    <add key="WSGI_ALT_VIRTUALENV_ACTIVATE_THIS" value="D:\home\site\wwwroot\env\Scripts\activate_this.py"/>
    <add key="WSGI_HANDLER" value="ptvs_virtualenv_proxy.get_virtualenv_handler()"/>
    <add key="WSGI_LOG" value="D:\home\LogFiles\EasternGroceryAppWCGI.log"/>
    <add key="PYTHONPATH" value="D:\home\site\wwwroot"/>
    <add key="APPINSIGHTS_INSTRUMENTATIONKEY" value="YOUR_APPINSIGHTS_KEY_HERE"/>
  </appSettings>
  <system.webServer>
    <modules runAllManagedModulesForAllRequests="true"/>
    <!-- customized activityTimeout (to 5 mins) and requestTimeout (to 60 mins)
        ref - https://www.iis.net/configreference/system.webserver/fastcgi
     -->
    <fastCgi>
      <application fullPath="D:\Python27\python.exe" arguments="D:\Python27\Scripts\wfastcgi.py" 
        maxInstances="4" idleTimeout="300" activityTimeout="300"
        requestTimeout="3600" instanceMaxRequests="10000" protocol="NamedPipe" flushNamedPipe="false">
      </application>
    </fastCgi>
    <handlers>
      <remove name="Python27_via_FastCGI"/>
      <remove name="Python34_via_FastCGI"/>
      <remove name="Python FastCGI"/>
      <add name="Python FastCGI" path="handler.fcgi" verb="*" modules="FastCgiModule" scriptProcessor="D:\Python27\python.exe|D:\Python27\Scripts\wfastcgi.py"
        resourceType="Unspecified" requireAccess="Script"/>
    </handlers>
    <rewrite>
      <rules>
        <rule name="Static Files" stopProcessing="true">
          <conditions>
            <add input="true" pattern="false"/>
          </conditions>
        </rule>
        <!-- below rule forwards all request to Python FastCGI handler - which inturn calls on 
        the WSGI_HANDLER - the Python Flask based EasternGrocery web app -->
        <rule name="Configure Python" stopProcessing="true">
          <match url="(.*)" ignoreCase="false"/>
          <action type="Rewrite" url="handler.fcgi/{R:1}" appendQueryString="true"/>
        </rule>
      </rules>
    </rewrite>
    <staticContent>
      <mimeMap fileExtension="." mimeType="text/plain"/>
    </staticContent>
  </system.webServer>
</configuration>
{% endhighlight %}

Changes to the web.config can be uploaded back to App Service through your deployment process or through `cURL` commands (see [invoking Azure Kudu API URL end points using `cURL`](kudu-curl)).

## `request`, `activity` and `idle` timeout for Java apps running in Tomcat or Jetty container in Azure App Service

App Service IIS supports `HttpPlatformHandler` directive that can be used to configure Tomcat or Jetty as an external process that can inturn run Java Web Apps. The [schema for the `HttpPlatformHandler` directive is available here](https://technet.microsoft.com/en-us/library/mt125371.aspx#Anchor_1). The `requestTimeout` can be changed via below `web.config` entry;

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<configuration xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">
  <system.webServer>
    <!-- unrelated to requestTimeout setting, below rewrite rule is to force HTTPS for all requests -->
    <rewrite>
      <rules>
	    <rule name="Force HTTPS" enabled="true" stopProcessing="true">
          <match url="(.*)" ignoreCase="true"/>
           <conditions>
            	<add input="{HTTPS}" pattern="off"/>
           </conditions>
          <action type="Redirect" url="https://{HTTP_HOST}{REQUEST_URI}" appendQueryString="true" redirectType="Permanent"/>
        </rule>
      </rules>
    </rewrite>
    <!-- sets request timeout to 4 mins -->
    <httpPlatform xdt:Transform="SetAttributes(requestTimeout)" requestTimeout="00:04:00"/>
  </system.webServer>
</configuration>
{% endhighlight %}

### Other limits

- If you have setup App Services Web Server (IIS) to invoke external process, you may have to watch out `startupTimeLimit` / `pingResponseTime` settings in the [processModel configuration](https://www.iis.net/configreference/system.applicationhost/applicationpools/add/processmodel#005)

- [28Mb max limit for the file uploads](https://www.iis.net/configreference/system.webserver/security/requestfiltering/requestlimits#005)
