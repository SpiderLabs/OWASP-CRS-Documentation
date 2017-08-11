===================================================
Rule IDs
===================================================

Why do rule IDs matter?
=======================
Each rule added to a ModSecurity instance must have a unique ID (in recent versions of modsec). As a result of this it is important that IDs be generated in a preplanned way such that they don't overlap with other rules. This becomes particularly important when it comes to rulesets, like OWASP CRS.
Because CRS can be used with other rule sets (in fact it is designed to be used with the Trustwave commercial rules, for instance) it is important that no two rulesets have overlapping ID ranges. To aid in this process the project maintains a list of reserved ID spaces. If you are planning on publishing rules or a ruleset please reach out to security[ at ]modsecurity.org and we can reserve a range of IDs for you thank you.

ID Reservations
---------------

* 1-99,999; reserved for local (internal) use. Use as you see fit but do not use this range for rules that are distributed to others.
* 100,000–199,999; reserved for rules published by Oracle.
* 200,000–299,999; reserved for rules published Comodo.
* 300,000-399,999; reserved for rules published at gotroot.com.
* 400,000–419,999; unused (available for reservation).
* 420,000-429,999; reserved for ScallyWhack .
* 430,000–439,999: reserved for rules published by Flameeyes
* 440,000-599,999; unused (available for reservation).
* 600,000-699,999; reserved for use by Akamai http://www.akamai.com/html/solutions/waf.html
* 700,000-799,999; reserved for Ivan Ristic.
* 900,000-999,999; reserved for the OWASP ModSecurity Core Rule Set project.
* 1,000,000-1,009,999; reserved for rules published by Redhat Security Team
* 1,010,000-1,999,999; unused (available for reservation)
* 2,000,000-2,999,999; reserved for rules from Trustwave's SpiderLabs Research team
* 3,000,000-3,999,999; reserved for use by Akamai 
* 4,000,000-4,099,999; reserved in use by AviNetworks
* 4,100,000-4,199,999; reserved in use by Fastly
* 4,200,000 and above; unused (available for reservation)


IDs in the OWASP CRS
====================

ID's within the OWASP Core Rule Set (CRS) have special meaning. Rules are assigned an ID based on their location within the ruleset. 
As the list above notes, the OWASP Core Rule Set is assigned ID's from 900,000 to 999,999. This means that each rule file in CRS 3.x has 1000 IDs reserved for it. Currently, this is more than enough space, however, if at some point this becomes a problem we'll cross that bridge when we come to it.
By design rules will always be separated by 10 IDs from the ID before it. The first 10 ID's (000 010,020,030,040,050,060,070,080,090) are designated for control flow related rules, that is typically rules that will trigger a skip action (these will be at the beginning of a file or rules that pass). 
As a result of this structure most rule files will start with ID 9[File_ID]100. As new rules are developed they should be added at the end. If it becomes necessary to add a rule in-between existing rules this must be noted in the comments. 

Mapping 3.0.0-dev(2.x) IDs to 3.0.0-rc1 IDs
===========================================
When we started developing CRS 3 we started with our old ModSecurity 2.x rules and used our experience to redesign the layout of the ruleset, refine rules that were failing, and add new controls that were needed. As a result of this in pre-release version of 3.x we ran into problems where rule IDs were all of the place. Not only was this confusing but it restricted the ability to do range based ID exclusions easily. As a result we re-structured the rule ID's to what we see above.

However, this is problematic for people who have made exceptions based on these existing rules. For these people we provide the following table:

See :download:`2.x to 3.x Rule ID Transition Table <_downloads/IdNumbering.csv>`.

You can use the above file along with a script similar to the one below to update exclusions or rule changes:

.. code-block:: python

	#!/usr/bin/env python
	# -*- coding: utf-8 -*-

	import csv
	import argparse
	import os
	import sys

	idTranslationFile = "IdNumbering.csv"

	if(not os.path.isfile(idTranslationFile)):
		sys.stderr.write("We were unable to locate the ID translation CSV (idNumbering.csv) please place this is the same directory as this script\n")
		sys.exit(1)
	parser = argparse.ArgumentParser(description="A program that takes in an exceptions file and renumbers all the ID to match OWASP CRS 3.0-rc1 numbers. Output will be directed to STDOUT and can be used to overwrite the file using '>'")
	parser.add_argument("-f", "--file", required=True, action="store", dest="fname", help="the file to be renumbered")
	args = parser.parse_args()
	if(not os.path.isfile((args.fname).encode('utf8'))):
		sys.stderr.write("We were unable to find the file you were trying to upate the ID numbers in, please check your path\n")
		sys.exit(1)
	fcontent = ""
	try:
		f = open((args.fname).encode('utf-8'), "r")
		try:
			fcontent = f.read()
		finally:
			f.close()
	except IOError:
		sys.stderr.write("There was an error opening the file you were trying to update")

	if(fcontent != ""):
		# CSV File
		f = open(idTranslationFile, 'rt')
		try:
			reader = csv.reader(f)
			for row in reader:
				fcontent.replace(row[0], row[1])
		finally:
			f.close()

	print fcontent
