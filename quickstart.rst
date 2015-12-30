===================================================
OWASP CRS Quickstart
===================================================

Welcome to the OWASP Core RuleSet (CRS) quickstart guide. We will attempt to get you up and running with CRS as quick as possible. This guide assumes ModSecurity is already working. If you are unsure see the [[install]] page. Otherwise, lets jump in.

You'll first need to download the ruleset. Using a browser (or equivalent) visit the following URL:

.. code-block:: bash

	wget https://github.com/SpiderLabs/owasp-modsecurity-crs/archive/master.zip

Once this is installed extract it somewhere well known on your server. Typically this will be in the webserver directory (we are demonstrating with Apache below)

.. code-block:: bash
	
	mv owasp-modsecurity-crs-master.zip /etc/httpd/
	cd /etc/httpd/
	unzip owasp-modsecurity-crs-master.zip -d owasp-modsecurity-crs


After extracting the rule set we have to set up the main OWASP configuration file. We provide an example configuration file as part of the package located in the main directory: modsecurity_crs_10_setup.conf.example. For many people this will be a good enough starting point but you should take the time to look through this file before deploying it to make sure it's right for your environment. For more information see [[Setting up the configuration file]]. 

Once you have changed any settings within the configuration file, as needed, you should rename it to remove the .example portion
	
.. code-block:: bash
	
	cd /etc/httpd/owasp-modsecurity-crs/
	mv modsecurity_crs_10_setup.conf.example modsecurity_crs_10_setup.conf
	
Only one more step! We now have to tell our web server where our rules are. We do this by including the rule configuration files. Again we are demonstrating using Apache but it is similar on other systems see the [[install]] page for details.

.. code-block:: bash
	
	echo 'IncludeOptional /etc/httpd/owasp-modsecurity-crs/modsecurity_crs_10_setup.conf' >> /etc/httpd/conf/httpd.conf
	echo 'IncludeOptional /etc/httpd/owasp-modsecurity-crs/rules/*.conf' >> /etc/httpd/conf/httpd.conf
	
Now that we have configured everything you should be able to restart and enjoy using the OWASP Core Rule Set. Typically these rules will require a bit of exception tuning, depending on your site. For more information see [[exceptions]]. Enjoy!

.. code-block:: bash
	
	systemctl restart httpd.service
	