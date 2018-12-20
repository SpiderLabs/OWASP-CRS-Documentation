=====================
Installing OWASP CRS
=====================
   
We are glad you chose OWASP CRS the premier free ModSecurity ruleset. Below you should find all the information you need to properly install CRS. If you are having problems feel free to reach out to our mailing list. You can signup at: https://lists.owasp.org/mailman/listinfo/owasp-modsecurity-core-rule-set
   
Prerequisites
=============
   
Installing the CRS isn't very hard but it does have one major requirement, ModSecurity. If ModSecurity isn't working properly you will likely run into problems running the CRS. In order to run the 3.x branch you require AT MINIMUM ModSecurity 2.8 or above, preferably version 3.x or above. 

Finding where to edit in your configuration
===========================================

ModSecurity comes in MANY different version with support for a multitude of Operating Systems (OS) and Web Servers. The installation locations may differ greatly between these different options so please be aware that the following are just some of the more common configurations.

Microsoft IIS with ModSecurity 2.x
----------------------------------

The most common deployment of ModSecurity for IIS is via the pre-packaged MSI installer, available at https://www.modsecurity.org/download.html. If you compiled or are looking to compile ModSecurity for IIS this documentation isn't for you. If you used this package to install ModSecurity 2.x on IIS (tested on IIS 7-10), than your configuration files are located within C:\\Program Files\\ModSecurity IIS\\ (or Program Files(x86) depending on your configuration). The inital configuration file, that is the one that the remainder are included from, is modsecurity_iis.conf. This file will be parsed by the ModSecurity for both ModSecurity and 'Include' directives.

By default all installations of ModSecurity without `SecRuleEngine <https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual#SecRuleEngine>`_ declared will start in DetectionOnly mode. IIS, by default, explitcly declares `SecRuleEngine <https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual#SecRuleEngine>`_ to be DetectionOnly within the included modsecurity.conf file. As a result any rule we make will only show up in the Windows Event Viewer by default. For our example we're going to turn on disruptive actions. To test we should add the following to the end of our modsecurity_iis.conf:

.. code-block:: bash

    SecRuleEngine On
    SecRule ARGS:testparam "@contains test" "id:1234,deny,status:403,msg:'Our test rule has triggered'"

This rule will be triggered when you go to your web page and pass the testparam (via either GET or POST) with the test value. This typically will looks similar to the following: http://localhost/?testparam=test. If all went well you should see an HTTP 403 Forbidden in your browser when you navigate to the site in question. Additionally, in your Event Viewer, under 'Windows Logs'->'Application', we should see a new log that looks like the following:

<Log.png>

If you have gotten to this step ModSecurity is functioning on IIS and you now know where to place new directives.

Apache 2.x with ModSecurity 2.x Compiled
----------------------------------------
Compiling ModSecurity is easy, but slightly outside the scope of this document. If you are interested in learning how to compile ModSecurity please go to the ModSecurity documentation. Having compiled ModSecurity there is a simple test to see if your installation is working. If you have compiled from source you would have needed to include 'LoadModule security2_module modules/mod_security2.so' either in httpd.conf (apache2.conf on Debian) or in some file included from this file. Anywhere after you load your module  you may add the following ModSecurity directives. 

.. code-block:: bash

    SecRuleEngine On
    SecRule ARGS:testparam "@contains test" "id:1234,deny,status:403,msg:'Our test rule has triggered'"

If you restart Apache you may now navigate to any page on your web server passing the parameter 'testparam' with the value 'test' (via post or get) and you should receive a 403. This request will typically appear similar to as follows: http://localhost/?testparam=test.

If you obtained a 403 status code your ModSecurity instance is functioning correctly with Apache and you now know where to place directives.


Apache 2.x with ModSecurity 2.x Packaged (RHEL)
-----------------------------------------------

Many operating systems provide package managers in order to aid in the install of software packages and their associated dependencies. Even though ModSecurity is relatively straight forward to install, some people prefer using package managers due to their ease. It should be noted here that many package managers do not up date their releases very frequently, as a result it is quite likely that your distribution may be missing required features or possibly even have security vulnerabilities. Additionally, depending on your package/package manager your ModSecurity configuration will be laid out slightly different.

On Fedora we will find that when you use 'dnf install mod_security' you will receive the base ModSecurity package. Apache's configuration files are split out within this environment such that there are different folders for the base config (etc/httpd/config/), user configuration (etc/httpd/conf.d/, and module configuration (/etc/httpd/conf.modules.d/). The Fedora ModSecurity 2.x package places the LoadModule and associated 'Include's within /etc/httpd/conf.modules.d/10-mod_security.conf. Additionally, it places some of the reccomended default rules in /etc/httpd/conf.d/mod_security.conf. It is this secondary configuration file that will setup the locations where you should add your rules. By default it reads in all config files from the /etc/httpd/modsecurity.d/ and /etc/httpd/modsecurity.d/activated_rules/ folder. To keep order I would reccomend testing this configuration by placing a rules.conf file within the activiated_rules folder. Within this rules.conf file add the following:

.. code-block:: bash

    SecRuleEngine On
    SecRule ARGS:testparam "@contains test" "id:1234,deny,status:403,msg:'Our test rule has triggered'"

Upon saving and restarting Apache (systemctl restart httpd.service) you should be able to navigate to a your local webserver. Once this is accomplished try passing the 'testparam' paramater with the value 'test' such as via the following URL:http://localhost/?testparam=test. You should receive a 403 Forbidden status. If you do congratulations, ModSecurity is ready for the OWASP CRS rules.

Nginx with ModSecurity 2.x Compiled
-----------------------------------
ModSecurity 2.x currently doesn't support the new Nginx loadable modules. As a result, it is required that you compile Nginx from source with ModSecurity. For more information on how to do this see the ModSecurity documentaiton. Once ModSecurity is compiled in you will have to specify both 'ModSecurityEnabled' and 'ModSecurityConfig' within any location block where you want ModSecurity enabled. An example would look similar to below.

.. code-block:: bash

    location / {
               ModSecurityEnabled on;
               ModSecurityConfig modsec_includes.conf;
           }

Within this modsec_includes you may use the Include directive to include other files or any ModSecurity directives. For our testing purpose we will add the following to our modsec_includes.conf:

.. code-block:: bash

    SecRuleEngine On
    SecRule ARGS:testparam "@contains test" "id:1234,deny,status:403,msg:'Our test rule has triggered'"
    
Upon saving and restarting Nginx (./nginx -s reload) you should be able to navigate to a your local webserver. Once this is accomplished try passing the 'testparam' paramater with the value 'test' such as via the following URL:http://localhost/?testparam=test. You should receive a 403 Forbidden status. If you do congratulations, ModSecurity is ready for the OWASP CRS rules.


Nginx with ModSecurity 3.x (libmodsecurity) Compiled
----------------------------------------------------
At current time of writing ModSecurity v3 is still in development. Please stay tuned for more information or visit the ModSecurity v3 repository at https://github.com/SpiderLabs/ModSecurity/tree/libmodsecurity 

Downloading OWASP CRS
=====================

Now that you know where your rules belong typically we'll want to download the OWASP CRS. The best place to get the latest copy of the ruleset will be from our Github: https://github.com/SpiderLabs/owasp-modsecurity-crs. Be careful to determine if there are any more relevant branches in development that can take advantage of the version of ModSecurity you are using. You can do this by checking the different branches on the site and looking throughout this documentation. To download a repository you can either click the '`Download Zip <https://github.com/SpiderLabs/owasp-modsecurity-crs/archive/master.zip>`_' button or your can use git clone. For instance, 

.. code-block:: bash

    git clone https://github.com/SpiderLabs/owasp-modsecurity-crs

Typically you'll end up with a folder named something similar to 'owasp-modsecurity-crs'. From here the process is surprisingly simple. Because OWASP CRS is, at its core, a set of ModSecurity configuration files (\*.conf files) all you have to do is tell ModSecurity where these CRS configuration files reside and it will do MOST of the remaining work. To do this you must use the 'Include' directive. This include directive can be used in similar places to where we used our SecRule earlier. It should be noted that OWASP CRS should be included AFTER the ModSecurity configuration rules which are available via the ModSecurity repo (at https://github.com/SpiderLabs/ModSecurity/blob/master/modsecurity.conf-recommended) which should have been configured during your inital installation. These rules will configure ModSecurity options, such as SecRuleEngine that we used earlier. This configuration file should be reveiwed and modified as desired.

Setup OWASP CRS
=====================
OWASP CRS contains one setup file that should be reviewed prior to completing setup. The setup file is the only configuration file within the root 'owasp-crs-modsecurity' folder and is named csr-setup.conf.example. Going through the configuration file (csr-setup.conf.example) and reading what the different options are is HIGHLY recommended. At minimum you should keep in mind the following.

* CRS does not configure ModSecurity features such as the rule engine, the audit engine, logging etc. This task is part of the ModSecurity initial setup.If you haven't done this yet please check out the recommended ModSecurity configuration at https://github.com/SpiderLabs/ModSecurity/blob/master/modsecurity.conf-recommended 
* By default (`SecDefaultAction <https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual#SecDefaultAction>`_) CRS will redirect to your local domain when an alert is triggered. This may cause redirect loops depending on your configuration. Take some time to decide what you want ModSecurity it do (drop the packet, return a status:403, go to a custom page etc.) when it detects malicious activity.
* Make sure to configure your anomaly scoring thresholds for more information see :doc:`anomaly`
* By default ModSecurity looks for lots of issues with different databases and languages, if you are running a specific environment, you probably want to limit this behaviour for performance reasons.
* ModSecurity supports Project Honeypot (http://www.projecthoneypot.org/index.php) blacklists. This is a great project and all you need to do to leverage it is sign up for an API key (http://www.projecthoneypot.org/httpbl_api.php)
* Do make sure you have added any methods, static resources, content types, or file extensions that your site needs beyond the basic ones listed.

For more information please see the page on :doc:`configuration`. Once you have reviewed and configured CRS you should rename the file suffix from .example to .conf

.. code-block:: bash

    mv csr-setup.conf.example csr-setup.conf

In addition to csr-setup.conf.example there are two other .example files within our repository. These files are: rules/REQUEST-00-LOCAL-WHITELIST.conf.example and rules/RESPONSE-99-EXCEPTIONS.conf.example. These files are designed to provide the rule maintainer the capability to modify rules (see :doc:`exceptions`) without breaking forward compatability with updates. As such you should rename these two files, removing the .example suffix. This will make it so that even when updates are installed they do not overwrite your custom updates. To rename the files in Linux one would use a command similar to the following:

.. code-block:: bash

    mv rules/REQUEST-00-LOCAL-WHITELIST.conf.example rules/REQUEST-00-LOCAL-WHITELIST.conf
    mv rules/RESPONSE-99-EXCEPTIONS.conf.example rules/RESPONSE-99-EXCEPTIONS.conf

    

Proceeding with the Install
===========================
Both ModSecurity 2.x (via APR) and ModSecurity 3.x support the Include directive and what it tells the ModSecurity core to do is parse the additional files for ModSecurity directives. But where do you place this folder for it to be included?
If you were to look at the CRS files, you'd note there are quite a few .conf files. While the names attempt to do a good job at describing what each file does additional information is available in the :doc:`rules` section. 

Includes for Apache
-------------------
Apache will include from the Apache Root directory (/etc/httpd/, /etc/apache2/, or /usr/local/apache2/ depending on the envirovment). Typically we reccomend following the Fedora practice of creating a folder specificlly for ModSecurity rules. In our example we have named this modsecurity.d and placed in within the root Apache directory. When using Apache we can use the wildcard notation to vastly simplify our rules. Simply copying our cloned directory to our modsecurity.d folder and specifying the appropertie include directives will allow us to install OWASP CRS. In the example below we have also included our modsecurity.conf file which includes reccomended configurations for ModSecurity

.. code-block:: bash

    <IfModule security2_module>
            Include modsecurity.d/modsecurity.conf
            Include modsecurity.d/owasp-modsecurity-crs/csr-setup.conf
            Include modsecurity.d/owsp-modsecurity-crs/rules/*.conf
    </IfModule>
    
Includes for Nginx
-------------------
Nginx will include from the Nginx conf directory (/usr/local/nginx/conf/ depending on the envirovment). Because only one 'ModSecurityConfig' directive can be specified within nginx.conf we reccomend naming that file modsec_includes.conf and including additional files from there. In the example below we copied our cloned owasp-modsecurity-crs folder into our Nginx configuration directory. From there we specifying the appropertie include directives which will include OWASP CRS when the server is restarted. In the example below we have also included our modsecurity.conf file which includes reccomended configurations for ModSecurity

.. code-block:: bash

    include modsecurity.conf
    include owasp-modsecurity-crs/csr-setup.conf
    include owasp-modsecurity-crs/rules/REQUEST-00-LOCAL-WHITELIST.conf
    include owasp-modsecurity-crs/rules/REQUEST-01-COMMON-EXCEPTIONS.conf
    include owasp-modsecurity-crs/rules/REQUEST-10-IP-REPUTATION.conf
    include owasp-modsecurity-crs/rules/REQUEST-11-METHOD-ENFORCEMENT.conf
    include owasp-modsecurity-crs/rules/REQUEST-12-DOS-PROTECTION.conf
    include owasp-modsecurity-crs/rules/REQUEST-13-SCANNER-DETECTION.conf
    include owasp-modsecurity-crs/rules/REQUEST-20-PROTOCOL-ENFORCEMENT.conf
    include owasp-modsecurity-crs/rules/REQUEST-21-PROTOCOL-ATTACK.conf
    include owasp-modsecurity-crs/rules/REQUEST-30-APPLICATION-ATTACK-LFI.conf
    include owasp-modsecurity-crs/rules/REQUEST-31-APPLICATION-ATTACK-RFI.conf
    include owasp-modsecurity-crs/rules/REQUEST-32-APPLICATION-ATTACK-RCE.conf
    include owasp-modsecurity-crs/rules/REQUEST-33-APPLICATION-ATTACK-PHP.conf
    include owasp-modsecurity-crs/rules/REQUEST-41-APPLICATION-ATTACK-XSS.conf
    include owasp-modsecurity-crs/rules/REQUEST-42-APPLICATION-ATTACK-SQLI.conf
    include owasp-modsecurity-crs/rules/REQUEST-43-APPLICATION-ATTACK-SESSION-FIXATION.conf
    include owasp-modsecurity-crs/rules/REQUEST-49-BLOCKING-EVALUATION.conf
    include owasp-modsecurity-crs/rules/RESPONSE-50-DATA-LEAKAGES-IIS.conf
    include owasp-modsecurity-crs/rules/RESPONSE-50-DATA-LEAKAGES-JAVA.conf
    include owasp-modsecurity-crs/rules/RESPONSE-50-DATA-LEAKAGES-PHP.conf
    include owasp-modsecurity-crs/rules/RESPONSE-50-DATA-LEAKAGES.conf
    include owasp-modsecurity-crs/rules/RESPONSE-51-DATA-LEAKAGES-SQL.conf
    include owasp-modsecurity-crs/rules/RESPONSE-59-BLOCKING-EVALUATION.conf
    include owasp-modsecurity-crs/rules/RESPONSE-80-CORRELATION.conf
    include owasp-modsecurity-crs/rules/RESPONSE-99-EXCEPTIONS.conf

Setting up automated updated
============================
todo:
The OWASP Core Rule Set is designed with the capability to be frequently updated in mind. New threats and techniques and updates are provided frequently as part of the rule set and as a result, in order to combat the latest threats effectivly it is imperative that constant updates should be part of your strategy.

An update script
----------------
As part of our continuing effort to provide the most user friendly rule set available we provide an example script that you can use for updating your ruleset:

.. code-block:: python

    # -*- coding: utf-8 -*-
    """
    This script is designed to allow users to automatically
    update their ModSecurity OWASP Core Rule Set. It can
    be called by a cronjob or scheduled task in order to
    allow for automation. Note that it can either replace
    the whole CRS directory or just update the rules folder,
    which is the default.
    """

    from __future__ import print_function
    import argparse
    import os
    import uuid
    import shutil
    import logging

    try:
        from git import Repo
    except ImportError:
        print("This script requires the GitPython module (pip install gitpython).")

    __author__ = "Chaim Sanders"
    __copyright__ = "Copyright 2016, Trustwave Inc"
    __credits__ = ["Chaim Sanders"]
    __license__ = "ASL 2.0"
    __version__ = "1.0"
    __maintainer__ = "Chaim Sanders"
    __git__ = "csanders-git"
    __status__ = "Production"

    def check_arguments(logger):
        """Control arguments and set args variable"""
        example_string = "Examples: python %(prog)s --full -p ./owasp-modsecurity-crs/" \
                         " or python %(prog)s --folder=util/ --path=./owasp-modsecurity-crs/util" \
                         " or python %(prog)s"
        parser = argparse.ArgumentParser(description='Update OWASP CRS rules',
                                         formatter_class=argparse.ArgumentDefaultsHelpFormatter,
                                         epilog=example_string)
        parser.add_argument('-b', '--branch', default="v3.0.0-rc1", type=str,
                            required=False, help='The GitHub branch you want to download.')
        parser.add_argument('-r', '--repo',
                            default="https://github.com/SpiderLabs/owasp-modsecurity-crs",
                            type=str, required=False, help='The GitHub repository you want to use.')
        parser.add_argument('-p', '--path', default="./owasp-modsecurity-crs/rules/", type=str,
                            required=False, help='The path where the rules files should be placed')
        group = parser.add_mutually_exclusive_group(required=False)
        group.add_argument('--full', action='store_true',
                           required=False,
                           help='Copy the whole repo to the path specified instead of just the rules')
        group.add_argument('--folder', default="rules/", type=str,
                           required=False,
                           help='The toplevel folder within the repo to copy. Can\'t be used with full')
        parser.add_argument('-d', '--debug', action='store_true',
                            required=False, help='Display debug logging.')
        args = parser.parse_args()
        if args.debug:
            logger.setLevel(logging.DEBUG)
            logger.debug("Debugging mode has been enabled.")
        logger.debug("The following arguments were assigned: " + str(args) + ".")
        return args

    def download_rules(args, logger):
        """Download and replace our rules"""
        if args.path[-1] != os.path.sep:
            dst_dir = args.path + os.path.sep
        else:
            dst_dir = args.path
        logger.debug("The final path was set to " + str(dst_dir) + ".")
        rand_fold = "./" + str(uuid.uuid4())
        logger.debug("The temporary repo folder was set to " + str(rand_fold) + ".")
        # If the user wants the whole directory set that otherwise just rules/
        if args.full:
            copy_fold = rand_fold
        else:
            copy_fold = rand_fold + os.path.sep + args.folder
        logger.debug("Set the folder to be copied to " + copy_fold + ".")
        Repo.clone_from(args.repo, rand_fold, branch=(args.branch))
        logger.debug("Cloned the repo successfully.")
        for src_dir, _, files in os.walk(copy_fold):
            for file_ in files:
                if file_[-8:] != ".example":
                    src_file = os.path.join(src_dir, file_)
                    # If its the full copy we need the new prefix path
                    if args.full:
                        copy_path = src_file.replace(rand_fold, "")[1:]
                    else:
                        # Remove the overlap in folders
                        copy_path = src_file.replace(os.path.join(rand_fold, args.folder), "")
                    dst_file = os.path.join(dst_dir, copy_path)
                    cwd = os.path.dirname(dst_file)
                    # Check if the directory structure exists
                    if not os.path.exists(cwd):
                        os.makedirs(cwd)
                    if os.path.exists(dst_file):
                        os.remove(dst_file)
                        logger.debug("Removed existing " + dst_file + ".")
                    try:
                        shutil.move(src_file, cwd)
                    except shutil.Error as exc:
                        print(exc)
                    logger.debug("Moved " + file_ + " to " + cwd + ".")
        logger.debug("Copying completed successfully")
        shutil.rmtree(rand_fold)
        logger.debug("Deleted the temporary folder.")

    def main():
        """Initiate logging and run subroutines"""
        logging.basicConfig(level=logging.INFO)
        logger = logging.getLogger(os.path.basename(__file__))
        args = check_arguments(logger)
        download_rules(args, logger)

    if __name__ == "__main__":
        main()

Problems with installation
==========================

Apache Line Continuation
------------------------
In Apache 2.4.x before 2.4.11 there is a bug where the use of line continuations in a config size may cause the line continuation to be truncated. This will lead to an error similar to the following:

.. code-block:: bash
	
    Syntax error on line 24 of /etc/httpd/modsecurity.d/activated_rules/RESPONSE-50-DATA-LEAKAGES-PHP.conf:
    Error parsing actions: Unknown action: \

This is not an error with ModSecurity or OWASP CRS. In order to fix this issue you can simply add a space before the continuation on the offending line. For more information see https://bz.apache.org/bugzilla/show_bug.cgi?id=55910    

Anamoly Mode Doesn't Work
-------------------------
Sometimes on IIS or Nginx users run into an instance where anamoly mode doesn't work as expected. In fact upon careful inspection of logs one would notice that rules don't fire in the order we would expect. In general this is a result of using the '*' operator within these envivornments as it does not act the same way as in Apache. In general within both Apache and IIS one should expliticly include the various files present within the OWASP CRS instead of using the '*'.

Webserver returns error after CRS install
-----------------------------------------
This is likley due to a rule triggering. For instance in some cases a rule is enabled that prohibits access via an IP address. Depending on your  `SecDefaultAction <https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual#SecDefaultAction>`_ and `SecRuleEngine <https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual#SecRuleEngine>`_ configurations, this may result in a redirect loop or a status code. If this is the problem you are experiencing you should consult your error.log (or event viewer for IIS). From this location you can determine the offending rule and add an exception if neccessary see :doc:`exceptions`.




