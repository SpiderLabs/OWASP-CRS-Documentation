=====================
Installing OWASP CRS
=====================

Contents:

.. toctree::
   :maxdepth: 1
   
   intro
   
We are glad you choose OWASP CRS the premier free ModSecurity ruleset. Below you should find all the information you need to properly install CRS. If you are having problems feel free to reach out to our mailing list. You can signup at: https://lists.owasp.org/mailman/listinfo/owasp-modsecurity-core-rule-set
   
Prerequisites
=============
   
Installing the CRS isn't very hard but it does have one major requirement, ModSecurity. If ModSecurity isn't working properly you will likely run into problems running the CRS. In order to run the 3.x branch you require AT MINIMUM ModSecurity 2.8 or above, preferably version 3.x or above. 

Finding where to edit in your configuration
===========================================

ModSecurity comes in MANY different version with support for a multitude of Operating Systems (OS) and Web Servers. The installation locations may differ greatly between these different options so please be aware that the following are just some of the more common configurations.

Windows IIS with ModSecurity 2.x
--------------------------------
By a LARGE majority the most common deployment of ModSecurity for IIS is via the pre-packaged MSI installer (If you compiled ModSecurity for IIS this documentation isn't for you). If you used this to install ModSecurity 2.x on IIS (tested on IIS 7-10), then your configuration files are located within C:\Program Files\ModSecurity IIS\modsecurity_iis.conf (this may be Program Files(x86) depending on your configuration). This modsecurity_iis.conf will be parsed by the ModSecurity core for ModSecurity and 'Include' directives.
By default all installations of ModSecurity without 'SecRuleEngine' started will start in DETECTIONONLY mode (which is pretty self explanatory). As a result any rule we make will only show up in the Windows Event Viewer by default. For our example we're going to turn on disruptive actions. To test we should add the following to our modsecurity_iis.conf:

SecRuleEngine On
SecRule ARGS:testparam "@contains test" "id:1234,deny,status:403,msg:'Our test rule has triggered'"

This rule will be triggered when you go to your web page and pass the testparam (via either GET or POST) with the test value. This typically will looks similar to the following: http://localhost/?testparam=test. If all went well you should see an HTTP 403 ModSecurity ACtion page in your browser when you navigate to the site in question. Additionally, in your Event Viewer, under 'Windows Logs'->'Application', we should see a new log that looks like the following:

<Log.png>

If you have gotten to this step ModSecurity functioning on IIS and you now know where to place new directives.

Apache 2.x with ModSecurity 2.x Compiled
----------------------------------------
Compiling ModSecurity is easy, but slightly outside the scope of this document. If you are interested in learning how to compile ModSecurity please go to the ModSecurity documentation. Having compiled ModSecurity there is a simple test to see if your installation is working. Typically if you compiled your own ModSecurity module you will not have split up your Apache configuration in any compilcated manner, often, all your configuration is located in httpd.conf. Anywhere after you load your module (typically done in httpd.conf by using 'LoadModule security2_module modules/mod_security2.so') you may add the following ModSecurity directives. 

SecRuleEngine On
SecRule ARGS:testparam "@contains test" "id:1234,deny,status:403,msg:'Our test rule has triggered'"

If you restart Apache you may now navigate to any page on your web server passing the parameter 'testparam' with the value 'test' (via post or get) and you should receive a 403. This request will typically appear similar to as follows: http://localhost/?testparam=test.

If you obtained a 403 status code your ModSecurity instance is functioning correctly with Apache and you now know where to place Directives.


Apache 2.x with ModSecurity 2.x Packaged
----------------------------------------

Many operating systems provide package managers in order to aid in the install of software packages and their associated dependencies. Even though ModSecurity is relatively straight forward to install, some people prefer using package managers due to their ease. It should be noted here that many package managers do not up date their releases very frequently, as a result it is quite likely that your distribution may be missing required features or possibly even have security vulnerabilities. Additionally, depending on your package/package manager your ModSecurity configuration will be laid out slightly different.
On Fedora we will find that when you use 'dnf install mod_security' you will receive the base ModSecurity package. Apache's configuration files are split out within this environment such that there are different folders for the base config (etc/httpd/config/), user configuration (etc/httpd/conf.d/, and module configuration (/etc/httpd/conf.modules.d/). The Fedora ModSecurity 2.x package places the LoadModule and associated 'Include's within /etc/httpd/conf.modules.d/10-mod_security.conf. Additionally, it places some of the reccomended default rules in /etc/httpd/conf.d/mod_security.conf. It is this secondary configuration file that will setup the locations where you should add your rules. By default it reads in all config files from the /etc/httpd/modsecurity.d/ and /etc/httpd/modsecurity.d/activated_rules/ folder. To keep order I would reccomend testing this configuration by placing a rules.conf file within the activiated_rules folder. Within this rules.conf file add the following:

SecRuleEngine On
SecRule ARGS:testparam "@contains test" "id:1234,deny,status:403,msg:'Our test rule has triggered'"

Upon saving and restarting Apache (systemctl restart httpd.service) you should be able to navigate to a your local webserver. Once this is accomplished try passing the 'testparam' paramater with the value 'test' such as via the following URL:http://localhost/?testparam=test. YOu should receive a 403 Forbidden status. If you do congratulations, ModSecurity is ready for the OWASP CRS rules. Proceed to....

Nginx with ModSecurity 2.x Compiled
-----------------------------------
TODO:

Nginx with ModSecurity 3.x Compiled
-----------------------------------
TODO: When using version 2.x with NGINX rules are specified in whatever ModSecurity configuration file you specified within your nginx.conf using the ModSecurityConfig directive. Within version 3.x rules can be specified anywhere after the *modsecurity* directive (which is typically in nginx.conf).

Proceeding with the Install
===========================
Now that you know where your rules belong typically we'll want to download the OWASP CRS. The best place to get the latest copy of the ruleset will be from our Github: https://github.com/SpiderLabs/owasp-modsecurity-crs. Be careful to determine if there are any more relevant branches in development that can more aptly take advantage of the version of ModSecurity you are using. You can do this by checking the different branches on the site and looking throughout this documentation. To download a repository you can either click the 'Download ZIP' button or your can use git clone. For instance, 

git clone https://github.com/SpiderLabs/owasp-modsecurity-crs

Typically when you either clone or download the zip you'll end up with a folder named something similar to 'owasp-modsecurity-crs'. From here the process is surprisingly simple. Because OWASP CRS is, at its core, a set of ModSecurity configuration files (*.conf files) all you have to do is tell ModSecurity where these CRS configuration files reside and it will do MOST of the remaining work. To do this you must use the 'Include' directive. This include directive can be used in similar places to where we used our SecRule earlier. Both ModSecurity 2.x (via APR) and ModSecurity 3.x support this directive and what it tells the ModSecurity core to do is parse the additional files for ModSecurity directives. 
But where do you place this folder for it to be included?
If you were to look at the CRS files, you'd note there are quite a few .conf files. While the names attempt to do a good job at describing what each file does additional information is available in the [[rules]] section. Fortunately, you don't manually have to include each and every configuration file. The include directive supports the wildcard character (*). Typically this means that you would 

Configuring CRS
===============
todo:

Setting up automated updated
============================
todo:

Problems with install?
======================
TODO: