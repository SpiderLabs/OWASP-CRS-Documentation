================================
Adding Exceptions and Tuning CRS
================================

Modifying Rules
===============

OWASP Core Rule Set (CRS) as a rule set for ModSecurity has no context into what your web application is doing and how it is designed. As a result, often legitimate requests might seem to CRS to be attacks. This term is often called a **False Positive**. While the CRS team does its best to prevent these from happening, they are somewhat inevitable due to the blacklist nature of the rules. A quick example might illustrate the problem better.

Imagine you are running the popular Wordpress CMS engine. As part of this engine the capability exists to add both HTML (and JavaScript if you're an administrator) to your blog posts. Now Wordpress has rules around which tags can be used and their whitelist of tags has generally been studied pretty heavily by the security community. However, ModSecurity will only has visibility akin to www.mywordpressblog.com?wp_post=<h1>Welcome+To+My+Blog</h1>. In this instance OWASP CRS sees HTML Injection, because that is what's there. ModSecurity will have no knowledge that this problem is mitigated server side and as a result may block the request. It is therefore necessary sometimes to add an exception.

The most common issues when adding exceptions to OWASP CRS is that if done 'inline' it will be clobbered by the next update. The ModSecurity team has developed sophisticated methods for dealing with this problem that are quite versatile. 

Exceptions versus Whitelist
---------------------------
There are two generally different methods for modifying rules. Exceptions, which will remove or modify the rule from startup time and whitelist modifications which can modify a rule based on the content of a transaction. In general whitelist rules are slightly more powerful but also more expensive as they must be evaluated every time a transaction comes in.

Within CRS 3.x two files are provided to help you add these different rule modifications, they are: rules/REQUEST-00-LOCAL-WHITELIST.conf.example and rules/RESPONSE-99-EXCEPTIONS.conf.example. As is noted in the :doc:`install` documentation, the .example extension is provided specifically so that when these files are renamed, future updates will not overwrite these files. As is listed within the :doc:`install` documentation, before adding a whitelist or exception modification you should rename these files to end in the .conf exception.

The naming of these files is also not an accident. Due to the transactional nature of the whitelist modifications, they need to take place BEFORE the rules they are affecting, since they are processed on every transaction. Conversely, the exceptions file contains directives that are evaluated on startup. As a result, these need to be some of the last rules included in your configuration such that data structures that store the rules are populated and it can modify them.

Writing Exceptions
------------------

Exceptions come in a few different forms which are all outlined in detail within the `Reference Manual <https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual>`_. The following directives can be used to modify a rule at startup without touching the actual rule:

* `SecRuleRemoveById <https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual#SecRuleRemoveById>`_
* `SecRuleRemoveByMsg <https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual#SecRuleRemoveByMsg>`_
* `SecRuleRemoveByTag <https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual#SecRuleRemoveByTag>`_
* `SecRuleUpdateActionById <https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual#SecRuleUpdateTargetById>`_
* `SecRuleUpdateTargetById <https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual#SecRuleUpdateTargetById>`_
* `SecRuleUpdateTargetByMsg <https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual#SecRuleUpdateTargetByMsg>`_
* `SecRuleUpdateTargetByTag <https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual#SecRuleUpdateTargetByTag>`_

You'll notice that there are two types of exceptions, those that remove, and those that change (or update) rules. General usage of the SecRuleRemove* rules is fairly straight forward:

.. code-block:: bash

    SecRule ARGS "@detectXSS" "id:123,deny,status:403"
    SecRuleRemoveById SecRuleRemoveByID 123

The above rule will remove rule 123. When ModSecurity starts up it will add rule 123 into its internal data structures and when it processes SecRuleRemoveById it will then remove it from it's internal data structures. The rule will not be processed for any transaction as by the time ModSecurity reaches the point where it's processing requests, the rule simply no longer exists. 

The SecRuleUpdate* modifications are a bit more complicated. They have the capability to update a target or action based on some identifier. The update target rule is perhaps the simpler of the two to use. Modifying Targets (Variables) is easy, you can either append or replace. To append you simply only list one argument after SecRuleUpdateTargetBy*. This is great for adding exceptions as you can restrict a certain index of a collection from being inspected  

.. code-block:: bash

    SecRule ARGS "@detectXSS" "id:123,deny,status:403"
    SecRuleUpdateTargetById 123 !ARGS:wp_post
    
It is also possible to replace a target (variable). To do this you must first list the variable you want to replace with, followed by the variable you want to replace. So in the below example we replace the ARGS variable with ARGS_POST.

.. code-block:: bash

    SecRule ARGS "@detectXSS" "id:123,deny,status:403"
    SecRuleUpdateTargetById 123 ARGS_POST ARGS


Updating an action becomes a little more tricky as there are default actions and actions types of which only one can exist per rule. In general, transformations and actions that are not already included will be appended. There is one big exception to this rule and that is disruptive actions (pass, deny, etc) will always replace each other, there may only ever be one disruptive action. Additionally, certain logging actions will replace each other, for instance nolog would overwrite the log action. This functionality has the same rules as using `SecDefaultAction <https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual#SecDefaultAction>`_. 

.. code-block:: bash

    SecRule "@detectXSS" attack "phase:2,id:12345,t:lowercase,log,pass,msg:'Message text'"
    SecRuleUpdateActionById 12345 "t:none,t:compressWhitespace,deny,status:403,msg:'New message text'"

    # Results in the following rule
    SecRule ARGS "@detectXSS "phase:2,id:12345,t:lowercase,t:none,t:compressWhitespace,log,deny,status:403,msg:'New Message text'"

In general updating a rule to remove just the false positive is preferred over removing the entire rule. It should be noted that both actions should be taken with care as they do open a potential security hole. Before you add an exception within any rules you should make sure that the area where you are adding the exception is indeed a false positive and not vulnerable to the issue.

You may notice that it is not possible to change the operator of the rule via these exception Directives. To change the functionality of a rule you must use whitelist modifications OR remove the rule and add a new one.

Writing Whitelist Modifications
-------------------------------

Whitelisting is more complicated than exceptions because the rules can be more varied. In some ways they are less powerful than exceptions, but in others they are far more powerful. Whitelist rules use the `ctl <https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual#ctl>`_ action to change the state of the engine on a per transaction basis. This can be as simple as turning off the ruleEngine when a certain IP hits. Note, the ruleEngine will return to state from the configuration file for the next transaction. 

.. code-block:: bash
    
    SecRule REMOTE_ADDR "@IPMatch 1.2.3.4" "id:1,ctl:ruleEngine=Off"
    
You can also use this rule to avoid certain rules in some cases, this effectively allows you to modify operators. In the following example we have a rule that will block the entire 129.21.x.x subnet (class B). We add a ctl modification before hand such that if we get a particular IP address in that range we remove the rule, effectively adding an exception

.. code-block:: bash

    SecRule REMOTE_ADDR "@IPMatch 129.21.3.17" "id:3,ctl:ruleRemoveById=4"
    SecRule REMOTE_ADDR "@IPMatch 129.21.0.0/24" "id:4,deny,status:403"
    
The `ctl <https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual#ctl>`_ action can also change the configuration of certain directives which can lead to more efficient rules. It is recommended that you investigate its full potential. 


Tuning CRS
==========

CRS 3.x is designed to make it easy to remove rules that are not relevant to your configuration. Not only are Rules organized into files that are titled with general categories but we have also renumbered according to a scheme such that rule IDs can be used to quickly remove entire unwanted configurations files by using `SecRuleRemoveById <https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual#SecRuleRemoveById>`_. The following example removes all XSS rules which are located in the REQUEST-41-APPLICATION-ATTACK-XSS.conf file. Notice that all OWASP CRS rules are prefixed with '9' and then the next two digits represent the rules file.

.. code-block:: bash
    
    SecRuleRemoveById "941000-941999" 

The CRS rules also features tags to identify what their functionality is. It is therefore easy to remove an entire category that doesn't apply to your environment. In the following example we remove all IIS rules using `SecRuleRemoveByTag <https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual#SecRuleRemoveByTag>`_.

.. code-block:: bash
    
    SecRuleRemoveByTag "platform-iis"

