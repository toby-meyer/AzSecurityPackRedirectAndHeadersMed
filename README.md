# Azure App Service Security Pack - Medium Strength

The purpose of this extension is to create a one-stop shop for common basic security concerns when using Azure Web Apps. 


## Basics

It is considered best practice when using Azure Web Apps or IIS to specify proper security headers and strip unnecessary headers. In addition, HTTPS everywhere is preferable while allowing Azure always-on to continue to work using keepalive requests. 

To accomplish this goal without installing several extensions per site along with custom applicationhost transforms, this extension was created to attempt to address many scenarios.

This version of the extension is "medium strength". Reasons for that classification are in the details below.  

## Details

This extension implements the following changes to your Azure Web App: 

* **HTTP to HTTPS Redirection with Keepalive Support**: This will redirect incoming non-TLS enabled requests on port 80 to port 443 using a 301 permanent redirect response which will negotiate TLS with the client. This implementation contains an exception for keepalive requests, both as identified as a warmup request by the server and user agent headers from the client containing initialization, sitewarmup, and always-on identifiers. This should account for most scenarios. [All the info](https://tools.ietf.org/html/rfc5246)

* **Strict Transport Security (HSTS)**: Strict transport security instructs the client browser to go directly to https even when http is specified for future visits. *includeSubdomains* (oddly) includes subdomains, *max-age* specifies this directive should live for a year, and *preload* requests that Google to set this directive in the shipping version of Chrome. [More Info](https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security)

* **Content Security Policy (CSP)**: This newer directive (relative to those below) defines proper client browser behaviors as it pertains to what type of actions can be honored for this site. This directive is why this particular extension is classified as "Medium Strength"; there are a few unsafe options specified to account for the somewhat common practice of using inline CSS, etc. These are the 'unsafe-inline'(x2), and 'unsafe-eval' parts of the directive. This will increase compatibility, but also *is not as secure* as with these directives omitted. If your site can operate without these directives, it is recommended to apply an additional transform to specify a CSP without them. Either way, however, the specification of any CSP is better than none. *Note*: The domain msecnd.net is allowed to facilitate the proper functioning of Microsoft Application Insights.  [More Info](https://content-security-policy.com/), [Testing Tools](https://report-uri.io/home/tools)

* **X-etc Headers**: X-Xss-Protection, X-Content-Type-Options, X-AspNet(Mvc)-Version, Referrer-Policy, and X-Frame-Options are directives that help to protect against cross-site-scrpting, version-specific attacks, and the like. Most are older than the CSP above, and thus supported by older browsers (*IE*). The X-Powered-By directive removes header information that serves no real purpose to the client and gives attackers information regarding the server that could be used to craft an attack, so we'll strip it. [More Info](https://www.owasp.org/index.php/OWASP_Secure_Headers_Project#tab=Headers)

## Installation

Install like any other Azure site extension by using the Azure portal (app service->extensions->add). In addition, you can install extensions with ARM templates as well. Here is an example of an this extension installed as a resource of a parent site named by the parameter "mysite", which also contains an appsettings stanza: 

        {
          "apiVersion": "2015-08-01",
          "name": "SecurityPackHttpsRedirectPlusHeadersMed",
          "type": "siteextensions",
          "dependsOn": [
            "[resourceId('Microsoft.Web/Sites', parameters('mysite'))]",
            "[concat(resourceId('Microsoft.Web/Sites', parameters('mysite')),'/config/appsettings')]"
          ]
        },

ARM Install Note: In my experience it's more reliable to set dependencies to force installation after all other parts of the app service are complete. In addition, the siteextensions.net site will timeout if too many extensions are requested in a short period of time. To address this in templates containing many sites, you can set dependsOn stanzas to reference extensions in a previously listed site to force synchronous processing. 

## Testing

After installing and restarting your site, you can test by inspecting the redirect, keepalive, and headers. To simplify the test process, tools such as [securityheaders.io](https://securityheaders.io/) can be used. As of this writing (August 2017) the changes made by this transform result in a security "grade" of "A" as reported on the aforementioned site. **Please note** that some sites **will** experience problems after installing this extension. Depending on your security needs it may be worth your time to update the security practices of your site, but if not simply delete this extension if issues are encountered. 

## Feedback 

Please feel free to open issues, etc. as you see fit. Thank you!