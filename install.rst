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

Windows IIS with ModSecurity 2.x
--------------------------------
By a LARGE majority the most common deployment of ModSecurity for IIS is via the pre-packaged MSI installer (If you compiled ModSecurity for IIS this documentation isn't for you). If you used this to install ModSecurity 2.x on IIS (tested on IIS 7-10), then your configuration files are located within C:\Program Files\ModSecurity IIS\modsecurity_iis.conf (this may be Program Files(x86) depending on your configuration). This modsecurity_iis.conf will be parsed by the ModSecurity core for ModSecurity and 'Include' directives.
By default all installations of ModSecurity without 'SecRuleEngine' started will start in DETECTIONONLY mode (which is pretty self explanatory). As a result any rule we make will only show up in the Windows Event Viewer by default. For our example we're going to turn on disruptive actions. To test we should add the following to our modsecurity_iis.conf:

.. code-block:: bash

    SecRuleEngine On
    SecRule ARGS:testparam "@contains test" "id:1234,deny,status:403,msg:'Our test rule has triggered'"

This rule will be triggered when you go to your web page and pass the testparam (via either GET or POST) with the test value. This typically will looks similar to the following: http://localhost/?testparam=test. If all went well you should see an HTTP 403 ModSecurity ACtion page in your browser when you navigate to the site in question. Additionally, in your Event Viewer, under 'Windows Logs'->'Application', we should see a new log that looks like the following:

<Log.png>

If you have gotten to this step ModSecurity functioning on IIS and you now know where to place new directives.

Apache 2.x with ModSecurity 2.x Compiled
----------------------------------------
Compiling ModSecurity is easy, but slightly outside the scope of this document. If you are interested in learning how to compile ModSecurity please go to the ModSecurity documentation. Having compiled ModSecurity there is a simple test to see if your installation is working. Typically if you compiled your own ModSecurity module you will not have split up your Apache configuration in any compilcated manner, often, all your configuration is located in httpd.conf. Anywhere after you load your module (typically done in httpd.conf by using 'LoadModule security2_module modules/mod_security2.so') you may add the following ModSecurity directives. 

.. code-block:: bash

    SecRuleEngine On
    SecRule ARGS:testparam "@contains test" "id:1234,deny,status:403,msg:'Our test rule has triggered'"

If you restart Apache you may now navigate to any page on your web server passing the parameter 'testparam' with the value 'test' (via post or get) and you should receive a 403. This request will typically appear similar to as follows: http://localhost/?testparam=test.

If you obtained a 403 status code your ModSecurity instance is functioning correctly with Apache and you now know where to place Directives.


Apache 2.x with ModSecurity 2.x Packaged
----------------------------------------

Many operating systems provide package managers in order to aid in the install of software packages and their associated dependencies. Even though ModSecurity is relatively straight forward to install, some people prefer using package managers due to their ease. It should be noted here that many package managers do not up date their releases very frequently, as a result it is quite likely that your distribution may be missing required features or possibly even have security vulnerabilities. Additionally, depending on your package/package manager your ModSecurity configuration will be laid out slightly different.
On Fedora we will find that when you use 'dnf install mod_security' you will receive the base ModSecurity package. Apache's configuration files are split out within this environment such that there are different folders for the base config (etc/httpd/config/), user configuration (etc/httpd/conf.d/, and module configuration (/etc/httpd/conf.modules.d/). The Fedora ModSecurity 2.x package places the LoadModule and associated 'Include's within /etc/httpd/conf.modules.d/10-mod_security.conf. Additionally, it places some of the reccomended default rules in /etc/httpd/conf.d/mod_security.conf. It is this secondary configuration file that will setup the locations where you should add your rules. By default it reads in all config files from the /etc/httpd/modsecurity.d/ and /etc/httpd/modsecurity.d/activated_rules/ folder. To keep order I would reccomend testing this configuration by placing a rules.conf file within the activiated_rules folder. Within this rules.conf file add the following:

.. code-block:: bash

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

.. code-block:: bash

    git clone https://github.com/SpiderLabs/owasp-modsecurity-crs

Typically when you either clone or download the zip you'll end up with a folder named something similar to 'owasp-modsecurity-crs'. From here the process is surprisingly simple. Because OWASP CRS is, at its core, a set of ModSecurity configuration files (*.conf files) all you have to do is tell ModSecurity where these CRS configuration files reside and it will do MOST of the remaining work. To do this you must use the 'Include' directive. This include directive can be used in similar places to where we used our SecRule earlier. Both ModSecurity 2.x (via APR) and ModSecurity 3.x support this directive and what it tells the ModSecurity core to do is parse the additional files for ModSecurity directives. 
But where do you place this folder for it to be included?
If you were to look at the CRS files, you'd note there are quite a few .conf files. While the names attempt to do a good job at describing what each file does additional information is available in the :doc:`rules` section. Fortunately, you don't manually have to include each and every configuration file. The include directive supports the wildcard character (*). Typically this means that you would 

Configuring CRS
===============
Going through the configuration file (modsecurity_crs_10_setup.conf.example) and reading what the different options are is HIGHLY recommended. At minimum you should keep in mind the following.

* CRS does not configure ModSecurity features such as the rule engine, the audit engine, logging etc. This task is part of the ModSecurity initial setup.If you haven't done this yet please check out the recommended ModSecurity configuration at https://github.com/SpiderLabs/ModSecurity/blob/master/modsecurity.conf-recommended 
* By default (SecDefaultAction) CRS will redirect to your local domain when an alert is triggered. This may cause redirect loops depending on your configuration. Take some time to decide what you want ModSecurity it do (drop the packet, return a status:403, go to a custom page etc.) when it detects malicious activity.
* Make sure to configure your anomaly scoring thresholds for more information see :doc:`anomaly`
* By default ModSecurity looks for lots of issues with different databases and languages, if you are running a specific environment, you probably want to limit this behaviour for performance reasons.
* ModSecurity supports Project Honeypot (http://www.projecthoneypot.org/index.php) blacklists. This is a great project and all you need to do to leverage it is sign up for an API key (http://www.projecthoneypot.org/httpbl_api.php)
* Do make sure you have added any methods, static resources, content types, or file extensions that your site needs beyond the basic ones listed.

For more information please see the page on :doc:`configuration`

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

Problems with install
=====================
In Apache 2.4.x before 2.4.11 there is a bug where the use of line continuations in a config size may cause the line continuation to be truncated. This will lead to an error similar to the following:

.. code-block:: bash
	
    Syntax error on line 24 of /etc/httpd/modsecurity.d/activated_rules/RESPONSE-50-DATA-LEAKAGES-PHP.conf:
    Error parsing actions: Unknown action: \

This is not an error with ModSecurity or OWASP CRS. In order to fix this issue you can simply add a space before the continuation on the offending line. For more information see https://bz.apache.org/bugzilla/show_bug.cgi?id=55910    
