==================
OWASP CRS Metadata
==================

OWASP CRS 3.x includes several pieces of metadata within rules. Not every rule features all metadata but it is a continuing project where possible to update this information to the best possible level. If you see a rule without a piece of metadata that you think is warranted please open either an issue request at https://github.com/SpiderLabs/owasp-modsecurity-crs/issues or a pull request. 

Tags about standards
--------------------

We try our best to outline information about the vulnerability when each rule triggers. To that end we include metadata about where that vulnerability type can be found in various standards. These tags are currently prefixed with the standard type and will be capitalized. They will NOT start with OWASP_CRS. They consist of a the standard and then a slash to indicate the start of the data. The following standards are supported:

* WASCTC
* OWASP_TOP_10
* PCI

Tags about paranoia level
-------------------------

Starting with version 3.0 we reintroduced the paranoia level concept. For tuning purposes we wanted it to be very clear which rules are from higher paranoia levels. As a result, any rule in a paranoia level greater than 1 will have a 'paranoia-level' tag. We use a forward slash to deliminate this tag. Anything after the slash should represent the paranoia level of that rule. For instance:

.. code-block:: bash

    tag:'paranoia-level/2'
    
Tags about rule classification
------------------------------

There are two different types of tags regarding rule classification. One type of tag carries over from CRS 2.x and one type has been added to CRS 3. The CRS 3 version can be more explanatory and allows for the capability to use SecRuleRemoveByTag more easily, whereas the CRS 2 versions give information in a more succinct manner and are used for anomaly scoring purposes.

The CRS 2.x rule classifications are indicated by tags starting with 'OWASP_CRS'. They are delimited by the forward slash (/) and will always have three parts. The sections get more specific as you progress through the string where the third section will always be the most specific. The following are a list of all the current CRS 2.x type tags that are used in CRS.

.. code-block:: bash

    tag:'OWASP_CRS/WEB_ATTACK/XSS'
    tag:'OWASP_CRS/AUTOMATION/SECURITY_SCANNER'
    tag:'OWASP_CRS/WEB_ATTACK/SQL_INJECTION'
    tag:'OWASP_CRS/LEAKAGE/SOURCE_CODE_JAVA'
    tag:'OWASP_CRS/LEAKAGE/ERRORS_JAVA'
    tag:'OWASP_CRS/POLICY/METHOD_NOT_ALLOWED'
    tag:'OWASP_CRS/LEAKAGE/ERRORS_SQL'
    tag:'OWASP_CRS/WEB_ATTACK/SESSION_FIXATION'
    tag:'OWASP_CRS/WEB_ATTACK/COMMAND_INJECTION'
    tag:'OWASP_CRS/WEB_ATTACK/DIR_TRAVERSAL'
    tag:'OWASP_CRS/WEB_ATTACK/FILE_INJECTION'
    tag:'OWASP_CRS/LEAKAGE/INFO_DIRECTORY_LISTING'
    tag:'OWASP_CRS/PROTOCOL_VIOLATION/INVALID_REQ'
    tag:'OWASP_CRS/PROTOCOL_VIOLATION/INVALID_HREQ'
    tag:'OWASP_CRS/PROTOCOL_VIOLATION/EVASION'
    tag:'OWASP_CRS/PROTOCOL_VIOLATION/MISSING_HEADER_HOST'
    tag:'OWASP_CRS/PROTOCOL_VIOLATION/MISSING_HEADER_ACCEPT'"
    tag:'OWASP_CRS/PROTOCOL_VIOLATION/MISSING_HEADER_UA'
    tag:'OWASP_CRS/PROTOCOL_VIOLATION/IP_HOST'
    tag:'OWASP_CRS/POLICY/SIZE_LIMIT'"
    tag:'OWASP_CRS/POLICY/ENCODING_NOT_ALLOWED'
    tag:'OWASP_CRS/POLICY/HEADER_RESTRICTED'
    tag:'OWASP_CRS/POLICY/FILES_NOT_ALLOWED'
    tag:'OWASP_CRS/PROTOCOL_VIOLATION/EVASION'
    tag:'OWASP_CRS/PROTOCOL_VIOLATION/MISSING_HEADER_ACCEPT'
    tag:'OWASP_CRS/POLICY/SIZE_LIMIT'
    tag:'OWASP_CRS/POLICY/EXT_RESTRICTED'
    tag:'OWASP_CRS/WEB_ATTACK/RFI'
    tag:'OWASP_CRS/WEB_ATTACK/REQUEST_SMUGGLING'
    tag:'OWASP_CRS/LEAKAGE/ERRORS_PHP'
    tag:'OWASP_CRS/LEAKAGE/SOURCE_CODE_PHP'
    tag:'OWASP_CRS/WEB_ATTACK/PHP_INJECTION'
    tag:'OWASP_CRS/LEAKAGE/ERRORS_IIS'

The CRS 3.x type tagging is split over a series of tags:

* application
* language
* platform
* attack

These tags all have additional information specified after a dash (-). In some cases, particularly with attack there might be another dash that represents a more specific variant for instance attack-injection-php. In general anything after the first dash can be treated as a string. If the category is general for a given rule the tag will have the data 'multi' an example would be a vulnerability that affects many different platforms. In this case the tag would look as follows:

.. code-block:: bash

    tag:'platform-multi'


Additional rule information
---------------------------

Often we will generate rules based on some presentation or article. In fact, sometimes the construction of the rule is done in such a way that it might not be naively clear how the rule works. In all of these cases comments will be left above the rule in question. Items like links will not appear within tag data.


