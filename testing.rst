=====================
Testing the Rule Set
=====================

Shipped with the ruleset are a number of different tools. One of these tools are the regression testing framework for OWASP CRS. For testing we use WTF (WAF Testing Framework). This framework allows us to specify HTTP requests via a YAML file. Each modification to CRS must pass the existing tests in order to be accepted. Additionally, if you are planning to add a rule, you should ensure that a test has been written. If you submit a pull request for a new rule without a test, it will not be accepted until one is provided.
