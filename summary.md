what is being discussed in the emails below and who is saying what and what is the conclusion: 

From: Patwardhan, Tanmay S <tanmay.s.patwardhan@ipmorgan.com>

Sent: Thursday, October 22, 2026 4:08 PM

To: Guo, Haiying <haiying.guo@chase.com>; Gopalakrishnan, Prasanth <prasanth.gopalakrishnan@ipmorgan.com> Cc: Mishra, Kailashchandra K <kailashchandra.k.mishra@ipmorgan.com»; Justa, Amit <amit.justa@ipmorgan.com>

Subject: Crawler strategy

@Guo, Haiying and @Gopalakrishnan, Prasanth

DataCompass currently uses Atlan crawlers- https://confluence.prod.aws.jpmchase.net/confluence/spaces/CTDATA/pages/4943462964/DataCompass+-+Atlan+Crawlers

Since we have divested from the Atlan Ul, and the Atlan contract is coming up for renewal in Feb 2027, we need a backup strategy for crawlers in case the terms of the contract negotiation are not sustainable long term
- Given the short time to Feb, we will need to continue with the Atlan contract for the crawlers, while we decide on a strategy and a roadmap for crawlers
- Whichever option we choose - we will need a level of customization in the crawling beyond what we have today since we are getting requests from CT towers and CDOs where customizations would help further automation of lineage
- Options are
I. Build our own - leverage inhouse expertise now in Rob Gubelic's team in Haiying's area
1. Partner with Steve M / Saurabh Gupta's AMDP team who owns JADE data harvesting
2. Ray also suggested to ask CCB whether they were using DataHub's open source crawlers
3. Continue with Atlan crawlers
Thoughts on which option to prioritize first pls?

Regards

Tanmay

Tanmay Patwardhan | Managing Director | Corporate Data Services | J.P. Morgan | 545 Washington Boulevard, Jersey City, NJ, United States | tanmay.s.patwardhan@jpmorgan.com | jpmorgan.com | JPMC Internal Use Only


RE: Crawler strategy

Guo, Haiying

To O Jusla, Ami; • Gopalakrishnen, Prasanth; • Palmardhan, Tanmay 5

• Mishra, Kailashchandra K

Enpits 4/20/2027

Boy- TBPB lineage is not crawlable. Believe me, we tried with IBM Menta and a number of different vendor tools.

From: Justa, Amit camit.justa@ipmorgan.coms

Sent: Thursday, October 22, 2026 5:05 PM

To: Guo, Haiying chaiying.guo@@chase.coms; Gopalakrishnan, Prasanth sprasanth gopalakrishnan@/ipmorgan.coms; Patwardhan, Tanmay 5 <tanmay.s.patwardhan@@ipmorgan.com

Cc: Mishra, Kailashchandra K <kailashchandra.k.mishra@jpmorran.com>

Subject: RE: Crawler strategy

A large part of our CT estate is on Oracle so would be worthwhile to invest time there. Crawling metadata may be simpler but mining Lineage is where we will have to drill more

From: Guo, Haiying <haiying.guo@chase.coms

Sent: Thursday, October 22, 2026 4:56 PM

To: Gopalakrishnan, Prasanth <prasanth.gopalakrishnan @ipmorgan.com»; Patwardhan, Tanmay 5 <tanmay.s.palwardhan@ipmorgan.com> Cc: Mishra, Kailashchandra K «kailashchandra.k.mishra@ipmorgan.com>; Justa, Amit <amit.justa@ipmorgan.com>

Subject: RE: Crawler strategy

Was just talking to Robert G. he offered his DBA expert to help research crawlers for OLTP databases. Let me know if you want to take him up for this.

From: Gopalakrishnan, Prasanth <prasanth.gopalakrishnan@ipmorgan.coms

Sent: Thursday, October 22, 2026 4:41 PM

To: Patwardhan, Tanmay S <tanmay.s.patwardhan@ipmorgan.com>; Guo, Haiying <haiving-guo@chase.coms Ce: Mishra, Kailashchandra K =kailashchandra.k.mishro@ipmorgan.coms; Justa, Amit <amit.justa@ipmorgan.coms

Subject: RE: Crawler strategy

Would say 1 and 3

1> Moden platforms(dbx, snowflake, sigma) this should be straight forward we can take 1 and Claude it up quickly to prove it out with robs team. more customizable to our needs

3> These were open source download.. just checked and able to pip install them. so available in JPMC and we can try this also.

@ 5 Reply

' Repyaa

From: Patwardhan, Tanmay 5 ctanmay.s.patwardhan@jpmorgan.coms

Sent: Thursday, October 22, 2026 4:08 PM

To: Guo, Haiying <haiving.guo@chase.com»; Gopalakrishnan, Prasanth <prasanth.gopalakrishnan@ipmorgan.com/g Ce: Mishra, Kailashchandra K «kailashchandra.k.mishra@ipmorgan.com»; Justa, Amit <amit.justa@ipmorgan.coms

Subject: Crawler strategy

@Guo, Haiying and @Gopalakrishnan, Prasanth