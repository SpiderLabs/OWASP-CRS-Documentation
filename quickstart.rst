===================================================
OWASP CRS Quickstart
===================================================

Welcome to the OWASP Core RuleSet (CRS) quickstart guide. We will attempt to get you up and running with CRS as quick as possible. This guide assumes ModSecurity is already working. If you are unsure see the :doc:`install` page. Otherwise, lets jump in.

You'll first need to download the ruleset. Using a browser (or equivalent) visit the following URL:

.. code-block:: bash

	wget https://github.com/SpiderLabs/owasp-modsecurity-crs/archive/master.zip

Once this is installed extract it somewhere well known on your server. Typically this will be in the webserver directory. We are demonstrating with Apache below For information on configuring Nginx or IIS see :doc:`install`. Additionally, while it is a successful practice to make a new 'modsecurity.d' folder as outlined below, it isn't strictly necessary. The path scheme outlined is the common to RHEL based operating systems, you may have to adjust the Apache path used to match your installation.

.. code-block:: bash
	
    mkdir /etc/httpd/modsecurity.d
    mv owasp-modsecurity-crs-master.zip /etc/httpd/modsecurity.d/
    cd /etc/httpd/modsecurity.d
    unzip owasp-modsecurity-crs-master.zip -d owasp-modsecurity-crs


After extracting the rule set we have to set up the main OWASP configuration file. We provide an example configuration file as part of the package (Note: Other aspects of ModSecurity are controlled by the recommended ModSecurity configuration rules, packaged with ModSecurity) located in the main directory: csr-setup.conf.example. For many people this will be a good enough starting point but you should take the time to look through this file before deploying it to make sure it's right for your environment. For more information see :doc:`configuration`. 

Once you have changed any settings within the configuration file, as needed, you should rename it to remove the .example portion
	
.. code-block:: bash
	
	cd /etc/httpd/modsecurity.d/owasp-modsecurity-crs/
	mv csr-setup.conf.example csr-setup.conf
	
Only one more step! We now have to tell our web server where our rules are. We do this by including the rule configuration files in our httpd.conf file. Again, we are demonstrating using Apache but it is similar on other systems see the :doc:`install` page for details.

.. code-block:: bash
	
	echo 'IncludeOptional /etc/httpd/owasp-modsecurity-crs/csr-setup.conf' >> /etc/httpd/conf/httpd.conf
	echo 'IncludeOptional /etc/httpd/owasp-modsecurity-crs/rules/*.conf' >> /etc/httpd/conf/httpd.conf
	
Now that we have configured everything you should be able to restart and enjoy using the OWASP Core Rule Set. Typically these rules will require a bit of exception tuning, depending on your site. For more information see :doc:`exceptions`. Enjoy!

.. code-block:: bash
	
	systemctl restart httpd.service
	
