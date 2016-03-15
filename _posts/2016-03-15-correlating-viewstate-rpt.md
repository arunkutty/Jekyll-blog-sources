---
layout: post
title:  Correlating ViewState and EventValidation in an RPT Script
date:   2016-03-15 18:29:00 +0530
categories: rpt correlation
---
**TLDR;** `Viewstate` and `EventValidation` may not be automatically correlated by [RPT](https://www.ibm.com/developerworks/downloads/r/rpt).
use the attached [rules file based on RPT 8.7.1.x]({{site.url}}/assets/VS_EV_Rules.zip) to correlate your tests in case your application uses them. And... `goto:` Disclaimer at the end of this note.

### Now for the story

So, I was in the lookout for a good start for my notes on Rational Testing tools. Since this was a minor win of last week,
I thought of jotting it down for the _posterity_ :wink:. If it becomes useful for someone, well, that's a pleasant side effect.

Correlation, a term test automation engineers may be well aware, in simple terms means picking 
up dynamic parts of server responses and substituting them in subsequent requests to the server. Using which a test automation tool mimics regular client software
such as a browser to effectively 'fool' the server. Dynamic parameters are inserted by Application servers as part of the workflow for a variety of reasons. 
ViewState and EventValidation are examples of such parameters that a server and client uses to exchange data in ASP.net. They are sometimes extremely long, making manual correlation 
cumbersome and error-prone.

RPT does most of the hardwork of correlation automatically. Auto-correlation, however, is on best-effort and there are chances for some misses. This [article](http://www.ibm.com/support/knowledgecenter/SSMMM5_8.7.1/com.ibm.rational.test.lt.common.doc/topics/cdatacorrfix.html) describes why some of them are missed. 

Correlating long-winding parameters such as ViewState and EventValidation through the GUI can be a tedious task especially if you have a large test with multiple occurences. 
A good option is to manually create one such reference and perform a substitution using it. After you are done, save the manual correlations to a Rules file. Now you can 'apply' these 
rules to the same test if you know that there are more of these parameters or in a different test altogether.

While creating a reference, RPT determines a suitable regular expression providing the user a provision to modify it if necessary. In this particular case, I modified the regular expression to extract ViewState to a simpler:<br>
`__VIEWSTATE\|(.*?)\|`
and EventValidation into this:<br>
`__EVENTVALIDATION\|(.*?)\|`<br>
This is how my rules file looked after these modifications:

<img src="{{ site.url }}/images/vs-ev-rules-snapshot.jpg" alt="Screenshot of the rules">


*Why personal win?* This must be at least the third instance where a customer case of playback failure due to these parameters ended up with me. I had so far been unsuccessful in 
getting the substitution part automated with rules. Using the references created automatically with the rules, I had been manually substituting them. A half-baked solution. 
Between the options I had i.e. 1. a one time effort to understand and fix the reason why my substitutions rule won't work and 2. Using "Datapool Candidates View" to manually
substitute, I had been picking the latter, which I hate to admit was the easier way out. So, this time around, I decided on the harder option to invest efforts to find out why 
substitutions don't work.

After a few trials, I found out what was going wrong. The substitution rule's "decode" flag was set to the default "false". In this case, the server was sending ViewState and 
EventValidation in URL Encoded form whereas they were in the decoded form in requests. Unless RPT is explicitly instructed to search for the "decoded" string by setting the
substitution's "decode" flag to "true", search of substitution sites will fail and no substitutions will be done. A trivial modification was all that was needed for a minor but 
satisfying win!


<img src="{{ site.url }}/images/substitution-decode-field.jpg" alt="screenshot of decode field">

Here is the final working [rule file]({{site.url}}/assets/VS_EV_Rules.zip) based on RPT 8.7.1.x. Thanks for reading and hope this note helps you some time!


**Disclaimer:** The rules file is being provided as-is. Feel free to modify to suit your needs.
