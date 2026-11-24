Summarize

00:00
We are running on rest API. So, in the migration, it actually took directly. And um?

  Speaker 1 00:14
I, I don't recall which warehouses for which workspace that's the Fallout right now, but I would need, uh, the warehouse IDs for each workspace for? Need to update saying that, hey, this is the. Oh, that was available for each workspace, and hence you can run on system tables so.

00:34
Uh, this might work. What product is the

  Speaker 2 00:39
Easiest way to my map? The warehouse ID to the crawler. This is the first time you need it for color, I think, right? So, yes, would be

  Speaker 1 00:46
The easiest way to figure out would be. Look at the corresponding, uh, minor workflow. Oh,

  Speaker 2 00:54
Okay.

  Speaker 3 01:00
Pages and all, but I think you already have. You can get to that information easily. Gottage, let me see Pokemon code. Okay, then that's only. I'll try it out if I'm not able to figure it out. I'll email you. Yeah, I think I mean, definitely. It is the same corresponding, just look for the same minor.

01:21
Yeah, 100 true, truly. Through it, India, they have been running fine. Another one, okay?

  Speaker 2 01:30
So, can we say that databricks is now? Like, past year, the SDR versions, the hyphen and HDR, was in a workflow that we were creating. They all passed. So, snowflake one, yes, data

  Speaker 1 01:46
Bricks. One didn't pass, but it was publishing we it, the wire mark. So, I mean, it was, the workflow was running for 23 hours. It was it published close to 1.6 million assets or something like that, so? We hope things work. Explained during the last phase, which is on our end.

02:06
So, SDR worked as a concept. And the extraction works and transmission works. Um, so that's where I started switching it up, saying that, hey, that's actually moves that config and migrate the actual required workflow because we know things work, so we should succeed. But yeah, we are yet to have a green run.

02:24
Wow, what was that last, like that, failed? You said?

  Speaker 2 02:31
The published one.

02:40
Yeah,

  Speaker 1 02:40
Completion time. Yeah, this was like a fresh workflow, right? So, it kind of was publishing a ton of assets. Yeah,

  Speaker 2 02:47
And it has the filtering. I mean, it was a brand new workflow for the whole workspace and had too much okay, okay, correct. So, I guess so. If you could safely assume that the publishing is happening? Exactly, so you wanted to go ahead. It does work because we already see assets and outline.

  Speaker 1 03:06
Um, I mean, it's already published 1.7 million assets 1.6 million added, so you know, it works. It just had to kind of finished, or it can increase the finish line assets. Yeah, that's awesome, and you cleaned up the new connection also, then hopefully. Not it, not it. Um, I'm vying for a green run, and once the other then.

03:27
Uh, clear it up. Yeah, the iPhone SDR connection. You will delete delete, right? Yeah, okay, yeah, great, uh, so that way we would have cleared snowflake and data breaks, uh? And I still have to go and get the Tableau thing started. Um. What does, huh? I wanted to discuss this thing with you.

  Speaker 2 03:50
Is it a good idea to go ahead and have? This SDR running parallel in production? With snowflake and data bricks. In production, and Tableau and third spot continue in production on the secure agent for a little while until we get through this. Uh, procurement thing. Um. Is that a good idea just so we can say something was accomplished by November 30th?

04:26
If at all that says hard date, somebody was tracking, and nobody has confirmed to me that there is some sort of a number 30th heart rate. Okay. Uh. So, what have you planned for productive weeks with them?

  Speaker 1 04:43
Um, would it be a different machine or the same machine, or what is the plan from your? Uh, machines are different. Two different va size, uh Dr and product?

  Speaker 2 04:55
Um, the deployment pipeline is ready available. That's what it was being used for for the uat deployments as well. Only thing that needs to be done. Fresh is creation of a new. Gsm lock box, which shouldn't be a big deal I see. Okay, okay, I understand. Then, let's take that route.

05:17
Um, let's not do something sort of a patchwork.

  Speaker 1 05:21
Um, I know, number 30 is a hard red line, even the other sports ID folks are kind of. Um. Struggling to meet that deadline? What? What was the deadline? When was it set? The deadline has has moved a lot in the world. Eileen July was a deadline to be honest with you.

05:44
Um, and yeah, that's that, I think, uh, some 1k1 got a negative. October was decided for you guys, and then October moved to November end. And yeah, I mean, I was actually going to ask you as well as true. I mean, are there any code, pieces, or holidays? Oh,

  Speaker 2 06:04
Definitely. Yes, like if if we didn't mess this November 30th, and in December, you'll hardly get like five days of open window. Yeah, until the end of first week of Jam. Yeah, yeah, I hear you do.

  Speaker 1 06:25
Tricky, so, and are you working? I mean, I think you were the primary point of contact so?

  Speaker 2 06:34
No, I met this, I believe, right, uh, primary. I'm communicating with you on a technical basis there. I have not been involving me on any of the communications with, like the management or.

  Speaker 1 06:48
You and I are talking in the capacity of technical, you know, correct work here.

  Speaker 2 06:56
So, I'm actually, uh, was a kind of unaware if there was a number 30th heart or soft or recommended. What kind of date it was? Started. I started coming to know about it a month ago, a month and a half or so ago.

  Speaker 1 07:14
Um, uh, so it is a hard design. But, and it is not completely on you as well, because we haven't kind of working through this. Yeah, so? A enchandro De Este 1, who will actually give the verdict that a planning supported or not, um? So, for other folks, leave your eyesight turning of WordPress schedules, saying that, hey, we cannot support it.

07:38
For the epmc, I think. We will want to support is my assessment, but I don't want to promise you that on this point this time. Yeah, like, I mean, thanks for, uh. Thanks for acknowledging that there was work to be done.

  Speaker 2 07:51
On your site as well. And obviously, I understand this was like, literally appearing, and onion, like one after the other one after. You wouldn't know what will come unless you feel that first layer.

08:03
Yeah,

  Speaker 1 08:04
So. Um, I don't know, I understand. So, I would say, let's. Keep going at it. And that's why I'm gonna ask him and or the question is to you that either you continuously working because I am working. Uh, I'm not taking off or anything, at least till they become the last two weeks or something.

08:24
I have exhausted my vacation for the year, I

  Speaker 2 08:27
Don't have any life. Yeah, I don't have no nothing left. In fact, myself days are also almost gone, and literally have two available cigarettes. Yeah. Hold on. Okay, that's yeah.

  Speaker 1 08:41
All right. So, yeah, let's just keep at it then. Um, because I believe there are a couple of steps left, like the? Um, migration, that's bringing up the box. I'm here to navigate through it, I think, from the desktop connectors we have hotspot. Left and top low left, right?

09:01
So I thought, yeah, I mean, if it's? Something of a secondary priority. We can definitely take. Take it up like that. Uh, agree that hey, data breaks snowflake. I completely understand. Tableau is also a priority, so shall we? Let's get that to you. Are you okay with us going to Prad, uh, trying to go to Prague with snowflake and databricks only and Str?

09:30
So that, so do you want to use the same box? Or yeah, it would be the same box. Yes, it needs to be the same box, so that's what I was going to ask that there will be like two run times there, then kubernetes, uh. Running the Tableau and thoughtspot, and then Padman running the databricks and.

  Speaker 2 09:48
No snowflake. So when you go live with SDR in production, we will have to turn off. Now, what is it? Yeah, is there a turn off need? Correct? I believe it's not a problem, but.

  Speaker 1 10:04
Um, everything's okay, I mean, you pick up the lock box and water because it might become a patchwork, and I don't want to kind of run into Rusty for private group creator. Yeah, so? You know, let's just push through, I would say. I think you're saying fine. Don't try to go to parallel mode or halfway in production.

  Speaker 2 10:26
Okay, let's see if the number of 30th gets negotiated if it was hard. I'm still unaware, but, uh, yeah. Ideal. I think your preference is KV to all of them all four at once, not halfway. Okay, so what I would say is that, I mean? Um, let's try to paralyze effort in the aspect that.

10:48
Um. Your procuring images for Tableau and thoughtsport for them. Yeah, uh.

  Speaker 1 10:53
And the same time, or in the meanwhile, if you have the box ready for Broad. If we can get the lock box ready, and here's the scripts that you made for. Dev. If that can be translated into fraud, that can be a parallel effort. And as soon as that's ready, we can have the snowflake and data bricks of start migrating over there sometimes.

11:15
Yeah, I'll I'll progress as if I were to migrate.

  Speaker 2 11:20
To get to proud by November 30th, but it is not extremely advisable we understand underneath, but we will progress with that in, I see. Okay, okay. Fairbal Lord, thanks! So that's? Yeah. And, and what about thoughtsport, though, as in, are you saying?

  Speaker 1 11:43
Dot support can be a secondary priority, or do we want?

  Speaker 2 11:46
I mean, personally, I think it can be a secondary plurality, but I will check. Yeah, what? What's the challenge in thought? Sorry, you can, uh, you can't recall the authentication. So that is correct. Exactly, so

  Speaker 1 11:58
Password, I believe, has three sort of authentication. And if I recall correctly, you depend on sample authentication, so similar to data breaks. There is ideally a code piece within our first file securation code on how to authenticate. There is some proxy box given by the passport team. Um, so wherever we hit, let's say, a particular URL like Dev URL.

12:23
Uh, the proxy helps authenticate with I think 80 anywhere once again and the auth, which is present. For Pottsport and get the token from the saml token from there. Uh, and then provides it to the workflow, so it is kind of similar to databricks and snowflake, which you have been doing.

12:47
But I mean for data breaks, and so, like, I mean it, it has been for database. Let's say it is packed over. Sorry, uh, for snowflake, it is oauth token, whereas for thoughts code, it is saml token.

13:06
So, the only problem is that, how do you authenticate and fetch that token, uh, for us, and put it in the secret? That's the challenge. Okay, yeah, I think, for me.

  Speaker 2 13:19
That achieving that by 30th is not possible for me, I think, but definitely freeway. Um, the challenge being, you'll have to unpack how the authentication works. Is that selling or? Uh, yeah, that that exactly is. The challenge is how to unpack what is happening right now, and converting it.

13:40
With another proxy Park, you know, service, or like, not sure how it could fit? Okay, now if I were to try and simplify that, um, and please write to look at it, said threats that advised. Uh, fresh pair of ice. Hmm. If you were to go to your internal documentation, right?

  Speaker 1 14:00
I believe databricks has its own page snowflake as its homepage, and that's where that has what I think Engineers, uh, something like that, uh, United, yeah, so? What other authentication mechanism we are using? Has it been provided in those documentation? Uh, where I'm coming from is that if there is a hot spot recommended way over there of authentication, can we or should we look into that?

14:27
And if that can easily approach that rather than? Trying to dig into what is happening. I'll try to do the need for my end as well. I see, I see, I think that's a good, uh.

  Speaker 2 14:38
Direction with idea on the bench. Okay, don't just try to replicate what you have right now, but see what's the ideal or supported, fully supported. We're currently by the engine 19. And then maybe we can just take that approach. Maybe that's already supported by your offering on the third spot connector for HDR.

  Speaker 1 14:57
Okay, exactly. So there might be because I believe there is some local proxy called hotspot, which is running already on the boxes there is. Yes, I don't know whether. Yeah, so if? We are just use that. Yeah, I mean, I mean, of course. We have the gift of AI right now.

15:15
You can just figure out how to authenticate using that, and that's the sample token. Um, that's all because the only difference is that arise. As far as I remember, is that? They never be hated Dev URL or test. You are a lot of broad URL. We used our code, had to hit a certain endpoint a different endpoint via the proxy for getting the for performing the authentication, and then getting the sample token.

15:45
So, we don't know how the proxy works at all, uh, R code, our only job. Or our code's job was to communicate to the proxy,

  Speaker 2 15:54
Saying that

  Speaker 1 15:55
If our request is from Dev. So, what to do for them? Ask the property the proxy. What do you know? At the end of the day, yeah, so? If that is complicated, we can always look for an easier message from that engineering engine.

16:17
As well. First, priority is to try and get ready for production release if it happens. Yeah, then come back to transport solution. Okay. For that second. Um, yeah, like, you said, get, uh, as your website already on Broad, right? Migrate snowflake and databricks.

16:49
Yeah, agreed. Effect. Okay, sounds good.

  Speaker 2 16:53
I think that's pretty much it, isn't it? For today, uh, I would like to see those workflow databricks workflows pass. Please, if you can click an eye, you know? Yeah, no, I am. I've been at it the whole day, and yeah, this is now when I realized that those are anyone that said, yeah, which was.

  Speaker 1 17:11
My bad nice, uh, yeah, Louis. I was just even I was using EA to automate a lot of things. Yeah, and it just happened to skipped over it. Yeah.

  Speaker 2 17:21
You know, you know, what? I came across a video yesterday and was talking about with the CIS support and all right, we will fall into. Cognitive surrender.

  Speaker 1 17:34
Oh well, yeah, yeah, that's cool. So that's I saw the example for my my own self also, and you mentioning that right now is another example. Yeah, no, no, of course, I said, reality.

17:49
Cool, yeah, like, so thank you. Yeah, I'll give you both.

  Speaker 2 18:03
I'll get to that today. Yes, thank you.

  Speaker 1 18:16
Okay, sure, just reminding yourself. Yes, I have not, but I will okay, sure.

  Speaker 3 18:22
Sounds good, nice. Thank you, bye.


SDR Migration – Status Report
As of November 24, 2026 | Target: November 30, 2026
Workstream	Status	Achieved	Pending / Blockers	Next Steps
Snowflake	Validated	SDR workflow successfully completed.	Production migration.	Migrate existing workflows to production SDR.
Databricks	Partially validated	Extraction, transmission and publishing proven. Approximately 1.6–1.7M assets published.	Green run pending; workspace-specific warehouse IDs required.	Map warehouse IDs from miner workflows, complete successful run and remove temporary test connection.
Tableau	Blocked	Preliminary SDR integration and troubleshooting performed.	Connector image procurement and validation.	Procure image, resolve outstanding issues and validate workflow.
ThoughtSpot	Blocked	Existing proxy-based SAML authentication mechanism identified at a high level.	Image procurement and SDR-compatible authentication.	Explore officially supported authentication mechanisms and confirm prioritization.
Production infrastructure	In progress	Dedicated PROD/DR VSIs and reusable deployment pipeline available.	Production GSM Lockbox and deployment configuration.	Create Lockbox, adapt UAT scripts and deploy SDR runtime.
Production migration	Pending	Migration approach discussed and aligned.	Connector readiness and production validation.	Prepare Snowflake and Databricks migration in parallel while resolving remaining connector blockers.
November 30 deadline	At risk	Deadline confirmed as hard.	All four connectors may not be ready. Limited December change windows.	Confirm deadline flexibility and ThoughtSpot prioritization with management.

Agreed Approach
- Parallelize production infrastructure preparation, workflow migration and connector remediation.
- Prioritize Snowflake, Databricks and Tableau.
- Avoid a fragmented production deployment with SDR and Secure Agent running concurrently.
- Continue targeting November 30 without compromising deployment stability.
Overall status: At risk
 Core SDR capabilities have been demonstrated, but full production readiness remains pending.

ThoughtSpot's secondary prioritization is proposed, not yet formally agreed.