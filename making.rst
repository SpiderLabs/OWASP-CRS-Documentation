=====================
Making Rules
=====================

The Basic Synatax
=================

A `SecRule <https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual#SecRule>`_  is a directive like any other understood by ModSecurity. The difference is that this directive is way more powerful in what it is capable of representing. Generally, a SecRule is made up of 4 parts:
* Variables - Instructs ModSecurity *where* to look (sometimes called Targets)
* Operators - Instructs ModSecurity *when* to trigger a match
* Transformations - Instructs ModSecurity *how* it should normalize variable data
* Actions - Instructs ModSecurity *what* to do if a rule matches

The structure of the rule is as follows:

.. code-block:: bash

    SecRule VARIABLES "OPERATOR" "TRANSFORMATIONS,ACTIONS"
    
A very basic rule looks as follows:

.. code-block:: bash

    SecRule REQUEST_URI "@streq /index.php" "id:1,phase:1,t:lowercase,deny"

The preceding rule will take each HTTP Request and obtain just the URI portion. From there is will transform the URI value to lowercase. Subsequently it will check to see if that transformed value is equal to exactly '/index.php'. If it is Modsecurity will deny the request, that is, it will stop processing further rules and intercept the request.

As can be seen from the previous explaination, one of the unique things about the SecRule directive is that each SecRule listed in your configuration is evaluated on each transaction. All the other ModSecurity directives are only evaluated at startup.

Clearly, if this was all there was to SecRules it wouldn't be very powerful. In fact, there is a lot more. So much more that it is in fact a full fledged language. The best place to find out about all the possible capabilities is via the `ModSecurity Reference Manual <https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual>`_. The following is just a glimpse of its capabilities:

There are ~105 **variables** in 6 different categories, some examples include:

* **Request Variables** - ARGS, REQUEST_HEADERS, REQUEST_COOKIES
* **Response Variables** - RESPONSE_HEADERS, RESPONSE_BODY
* **Server Variables** - REMOTE_ADDR, AUTH_TYPE
* **Time Variables** - TIME, TIME_EPOCH, TIME_HOUR
* **Collection Variables** - TX, IP, SESSION, GEO
* **Miscellaneous Variables** - HIGHEST_SEVERITY, MATCHED_VAR  

There are ~36 **operators** in 4 different categories, some examples include:

* **String Operators** - rx, pm, beginsWith, contains, endsWith, streq, within
* **Numerical Operators** - eq, ge, gt, le, lt
* **Validation Operators** - validateByteRange, validateUrlEncoding, validateSchema
* **Miscellaneous Operators** - rbl, geoLookup, inspectFile, verifyCC


There are ~35 **transformation** functions in 6 different categories, some examples include:

* **Anti-Evasion Functions** - lowercase, normalisePath, removeNulls, replaceComments, compressWhitespace
* **Decoding Functions** - base64Decode, hexDecode, jsDecode, urlDecodeUni
* **Encoding Functions** - base64Encode, hexEncode
* **Hashing Functions** - sha1, md5

There are ~47 **actions** in 6 different categories, some examples include:

* **Disruptive Actions** - block, drop, deny, proxy
* **Flow Actions** - chain, skip, skipAfter
* **Metadata Actions** - phase, id, msg, severity, tag
* **Variable Actions** - capture, setvar, initcol
* **Logging Actions** - log, auditlog, nolog, sanitiseArg
* **Miscellaneous Actions** - ctl, multiMatch, exec, pause, append/prepend

While there are a lot of options available there are a few basic things to remember.

1. Every SecRule must have a VARIABLE
2. Every SecRule must have an OPERATOR, if none is listed @rx is implied.
3. Every SecRule must have an ACTION. The only required action is id, however, several actions are implied by SecDefaultAction (default phase:2,log,auditlog,pass)
4. Every SecRule must have an phase ACTION, this tells the rule when to fire. If no phase is included the default is phase:2.
5. Every SecRule must have a disruptive ACTION. This is an action that describes what to do with the transaction if triggered. If no disruptive action is included the default is pass
6. Transformations are optional but should be used to prevent your rule from being bypassed


Advanced Variable Usage
=======================

Variables themselves are quite easy to access as our earlier rules have shown. There are a couple of corner cases though. Not all variables are strings. Some variables, like ARGS_GET are **Collections**. A collection is very similar to a dictionary in Python, it is a key value pair of information. For instance if our request URI was http://www.example.com?x=test&y=test2 our collection might look like ARGS_GET = {"x" : "test", "y" : "test2"}. When we request an VARIABLE that is a collection, ModSecurity will iterate over each value in the collection, applying transformations and checking against the operator, until it finds a match. If it finds a match it will stop processing the rule and undertake any actions specified.

.. code-block:: bash

    SecRule ARGS_GET "@contains test" "id:1,phase:1,t:lowercase,deny"

It is possible to access just an index of a collection as well. This makes addressing specific variable very easy. To do this within the VARIABLE area of a SecRule you use the ':' (colon).

.. code-block:: bash

    SecRule ARGS_GET:username "@contains admin" "id:1,phase:1,t:lowercase,deny"   

One operator is nice but what if I have to apply one operator on multiple rules, for instance I want to check GET parameters and COOKIES. ModSecurity provides a way for you to do this. You can use the '|' (pipe) to combine two VARIABLES into one rule. This pipe can be applied as many times as you want. In the example below we combine both GET and POST arguments. In fact, this is not necessary in reality as there is a built in ARGS collections that already does this.

.. code-block:: bash

    SecRule ARGS_GET|ARGS_POST|REQUEST_COOKIES "@rx hello\s\d{1,3}" "id:1,phase:2,t:lowercase,deny"
    
If you are having a problem where one of your variables is causing false positives or you just don't want to look there you can also remove an index of a collection using the '!' (exclamation mark). This almost always used in conjunction with the pipe and an index, ModSecurity will understand that this means remove this index from the collection

.. code-block:: bash

    SecRule ARGS|!ARGS:password "@rx (admin|administrator)" "id:1,phase:2,t:lowercase,deny"
    
Advanced Transformation Usage
=============================
The concept of transformations is very intuitive and thanks to ModSecurity's open source nature there are quite a few to choose from. An issue arises in that the proper application of transformations often takes knowledge about how the threat you are trying to stop can manifest itself. Imagine the following example - you are trying to detect an XSS (Cross Site Scripting) attack. 

Your first attempt looks like the following:
    
.. code-block:: bash

    SecRule ARGS "@contains <script>" "id:1,deny,status:403"

This can be easily bypassed by using uppercase such as ?x=<sCript>alert(1);</script>. So we can apply a transformation:

.. code-block:: bash

    SecRule ARGS "@contains <script>" "id:1,deny,status:403,t:lowercase"

This too can be easily bypassed by simply appending a space ?x=<sCript >alert(1);</script>. So we need more transformations:

.. code-block:: bash
    
    SecRule ARGS "@contains <script>" "id:1,deny,status:403,t:lowercase,t:removeWhitespace"

In many contexts HTML entities might be interpreted and converted back to their ASCII form. That would allow us to by pass this rule with something like &lt;sCript >alert(1);</script>. 

.. code-block:: bash
    
    SecRule ARGS "@contains <script>" "id:1,deny,status:403,t:lowercase,t:removeWhitespace,t:htmlEntityDecode"

As you can see this type of approach can go on for a while and is why OWASP CRS is important. We put our rules out there and continuously allow people to test them. If they find an issue we fix it and the cycle continues. This attempt to blacklist malicious attacks is a constant battle, it is always encouraged that you whitelist where available. Currently if you look at CRS you'll see there are many XSS rules. Each rule may have 5 or 6 different transformations and the operators get more complex all the time.

Advanced Action Usage
=====================


Advanced Operator Usage
=======================
Most OPERATORS are self explanatory. Many operators such as the string manipulation operators take arguments. Some operators such as the libinjection @detectXSS do not. In general the semantics for most OPERATORS is quite straight forward. The exception to this rule is the default operator @rx, or regular expressions. Even if you know how to use regular expression, it is still quite easy to make a mistake in a security context. When writing a rule remember the following guidelines:

* Regexp should avoid using ^ (alternative: \A) and $ (alternative: \Z) symbols, which are metacharacters for start and end of a string. It is possible to bypass regex by inserting any symbol in front or after regexp.
* Regexp should be case-insensitive. It is possible to bypass regex using upper or lower cases in words. Modsecurity transformation commands (which are applied on string before regex pattern is applied) can also be included in tests to cover more regexps [51].
* Regexp should avoid using dot “.” symbol, which means every symbol except newline (\n). It is possible to bypass regex using newline injection.
* Number of repetitions of set or group {} should be carefully used, as one can bypass such limitation by lowering or increasing specified numbers.
* Best Practice from slides of Ivan Novikov [2]: Modsecurity should avoid using t:urlDecode function (t:urlDecodeUni instead).
* Regexp should only use plus “+” metacharacter in places where it is necessary, as it means “one or more”. Alternative metacharacter star “*”, which means “zero or more” is generally preferred.
* Usage of wildcards should be reasonable. \r\n characters can often be bypassed by either substitution, or by using newline alternative \v, \f and others. Wildcard \b has different meanings while using wildcard in square brackets (has meaning “backspace”) and in plain regex (has meaning “word boundary”), as classified in RegexLib article [42].
* Regexp should be applied to right scope of inputs: Cookies names and values, Argument names and values, Header names and values, Files argument names and content.
* Regular expression writers should be careful while using only whitespace character (%20) for separating tag attributes. Rule can be bypassed with newline character: i.e. %0d,%0a.
* Greediness of regular expressions should be considered. Highlight of this topic is well done in Chapter 9 of Jan Goyvaert’s tutorial [27]. While greediness itself does not create bypasses, bad implementation of regexp Greediness can raise False Positive rate. This can cause excessive log-file flooding, forcing vulnerable rule or even whole WAF to be switched off.

Rules for CRS
=============
All rules for CRS should include at least one regression test. To increase the chances of having your pull request accepted into the mainline more regression tests are recommended.

If your rule contains combination of data sources into a single regular expression for performance reasons you should document the use of the regexp-assemble command in the comments above your command. You should also include your independent sources within this util directory. Doing so increases overall maintainability. 