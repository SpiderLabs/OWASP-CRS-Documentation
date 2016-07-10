=====================
Making Rules
=====================


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

