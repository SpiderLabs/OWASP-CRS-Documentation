Welcome to OWASP CRS Documentation's documentation!
===================================================

Contents:

.. toctree::
   :maxdepth: 2
   
   intro
   
Rules are assigned an ID based on their location within the ruleset. The OWASP Core Rule Set is assigned ID's from 900,000 to 999,999. Each file has 1000 ID's reserved for it. If that is not enough space, we'll cross that bridge  By default new rules will always be separated by 10 ID's from the ID before it. The first 10 ID's (,000 010,020,030,040,050,060,070,080,090) are designated for control flow related rules, that is typically rules that will trigger a skip action (these will be at the beginning of a file or rules that pass). Therefore most rule files will start with ID 9[File_ID]100. New rules should be added at the end. If it becomes necessary to add a rule in-between existing rules this must be noted in the comments.   