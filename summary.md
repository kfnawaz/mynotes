Summarize the objective, key points discussed, options, and next steps. 

00:00
Good morning, I'm good. How are you? I'm doing great as well. Unfortunately, stuck once again.

  Speaker 1 00:14
Let's, let's give it a minute. Tips to see if, uh, okay? Galax can join or not? All right, okay?

00:45
All right, everything.

00:57
Let's go ahead and get started. That's fine then. So, you have the information about all things that are happening here, right? So, we.

  Speaker 2 01:14
Do you agree with the challenges associated with the AWS? Do you have a full understanding of it? What what these guys are saying, pay. I think what they're saying is just the pattern itself is not allowed. Forget the technical challenge of getting the.

  Speaker 1 01:31
Authentication mechanism working on the vsi, the missing piece.

  Speaker 2 01:36
Don't even do that. I think it was the message. I honestly didn't understand what they are complaining about. Um, because I, yeah.

  Speaker 3 01:51
Yeah, I mean.

  Speaker 1 01:52
I'll be honest. I haven't read every word of that email. Okay, that's fine. Let's, let's go over it. I'll share my screen here. Yeah, I don't even I have that music open. And I could see a few lines from James. Let me support. Yeah, I don't know why. You're asking something like that, so?

02:13
I could tell, or I could understand that. They don't want to do it. But they do it a lot, I don't know. Uh, yeah, so?

  Speaker 2 02:27
Is saying these two sentences, basically I from the architecture cyber security pattern? Access to the secret manager outside a VPC is considered a security risk, which is exactly what we are trying to do right. Accessing a secret manager outside the? Jbmc. Hmm, um, no, no, not outside VPC is yours, so?

  Speaker 1 02:53
I, I think what they can actually support is a BM in the VPC, but your machine is elsewhere. Whether it's a bare metal one, so it is not in that Network. Uh, region or not. Region is not the right word. The network boundary. And once, I think they're discouraging it, I.

03:15
Oops! Yeah, I mean, I, I saw a question as well, I mean, bypassed, or something like that, but? I don't know what the answer was for that. Yeah, he basically directed us to go to the IIM product managers after that, right after I like insisted asking some more about it?

03:37
The use of the secret manager.

  Speaker 3 03:42
Outside ABS AWS is a hard control boundary.

  Speaker 1 03:47
Yeah, but I am. And yeah. Means for each I am or per. I am team,

  Speaker 3 03:56
Uh oh, I am team. It means, okay,

  Speaker 1 03:58
Okay, okay,

  Speaker 3 03:59
I am got it. Okay, I think what they mean here is if your deployment is enabius AWS. Use AWS Secrets manager, but if your deployment is runtime is somewhere else don't use AWS. I think that's what he meant here.

  Speaker 1 04:17
Yeah, that's what you and I could understand.

  Speaker 3 04:22
Um, so we have. We still, he kept the heater open to talk to these product owners and see. Uh, but likely it is a hard stop. Yeah, right. Like, I could tell, yeah.

  Speaker 1 04:39
It's like, we're testing our blood or exception. Yeah, we could possibly test our luck for exception, but we'll see. Then comes the S the GSM part, right? So GSM, yes, I meant.

  Speaker 3 04:53
The documents that I sent to you yesterday when I said from earlier in the email, I don't know, did I reply? I didn't, I replied.

  Speaker 1 05:01
So I got one document? Yes. Yeah, that, uh?

  Speaker 3 05:12
The GSM. All that I thought was relevant for your integration point of view. Hopefully that will help. Uh. And.

05:25
The third thing that I wanted to ask before I forget is the possibility of actually going back to kubernetes right. Is there a possibility that we can just set it up in kubernetes, because overall, the problem is the a secret store. Being being either outside. On-Prem, or if it is GSM, then you have to do a product change on your side.

05:50
What if we is there a way to have this SDR run in the existing k3s setup that we have, but just a different namespace.

  Speaker 2 06:01
The secret store is already in there. The token generator, which refreshes the secret store every few, so every so often is also in place.

  Speaker 3 06:09
Is that a possibility? So, the technical answer is yes.

  Speaker 1 06:15
But I. We will encourage that because we have faced multiple problems already with k3s, right? So, yeah, I understand

  Speaker 3 06:25
That the kubernetes Secrets part is something which you can use. I mean, we have a sort of alternative over here, like you can use environment variables. Uh, which will stay locally on the machine. Um, but I don't know whether that is acceptable. To you or not. You as?

  Speaker 1 06:43
Your team. So, you mean,

  Speaker 2 06:46
Uh, our token generator jobs are also workloads within the kubernetes, right? Thanksgiving, yeah, okay, you remember, uh, how we have it set up the? Your secrets are in the kubernetes secret store. But then they change over time, right? Some of especially the snow, correct times. They change over time, so we have a separate workloads like the crown jobs, uh, that run certain, you know, certain repo and gets refreshes the token.

07:18
And updates this kubernetes secret store every 30 minutes or so. Correct. Okay, so what if we have? That running as is and in? Instead, maybe change the environment. Variables like, will, so for okay. I'm losing words here, so our current K3s has secure agent. Uh, in certain containers, and then two can generated jobs in certain containers.

07:48
It will get rid of the secure agent containers. And leave the set token generator containers as is in the currents k3s setup, and those could token generated jobs can. Refresh it, but then instead of populating the k3s secret store, they can go populate the environment variables in HDR.

08:14
Is that possible?

08:17
Outside. Then there would be drinking outside k3s environment.

08:24
So? Again. Um, you know, outside previous environment. Technically speaking. Yes, because it's running on that machine itself, so it should be able to.

  Speaker 1 08:36
Um, kind of stupid. Now, let's give me one second, just take care of. Sure, give me this literally two seconds. Yeah, sure.

09:56
So, yeah, she actually got. I mean, I ring at Poland.

10:06
Yeah, I said 10 minutes, but she reset the counter, and then I'll watch people I had to give her a piece of my mind,

  Speaker 3 10:13
Oh my God. It was so little man, you give peace of Mind. I know, not the right choice of words.

  Speaker 1 10:30
Uh, but yeah, sorry. So what you're suggesting is using the same Chrome job in K3. Yes, well, I didn't. Humanities shouldn't matter. I mean, a cron job is a cruncher by the end of the day. Um, I know you are putting it in kubernetes Secrets, but?

10:56
Um, module in it, so you can definitely reuse that, right? Um. I mean, even if you're using kubernetes, I I definitely. Encourage having further dependency on secure agent or the k3s, for that matter.

11:18
Then we learn into troubles. Whenever let's say something breaks, yeah, you don't have any experts, or um, I can try the best. Yeah, yeah, there will be ownership problems like and support. Our support team cannot handle that, right? They're not experts at all. I understand and. Yeah,

  Speaker 3 11:36
But your point basically is that as long as you have that, we have the token generator repo that can run somewhere else. Also not inside kubernetes. Just exactly, okay, fine. But can that, if it's running just outside on, uh, on the on the same vsi? That could. And that could keep the environment's variables updated, right?

11:57
Every 30 minutes. Will that work? Yeah.

  Speaker 4 12:05
So again, let me when I was thinking as to what will happen. The answer is yes, you can definitely do that.

12:15
Here comes a secondary problem to it where?

  Speaker 1 12:20
When the container of the docker container is running, it reads a static file like environment variable. Let's say, XYZ were there. Yeah,

  Speaker 2 12:27
You

  Speaker 1 12:28
Basically had updated. The container is up exactly. The containers came up. It knows XYZ and its values. But let's say, if you updated ABC or updated the value of XYZ, there is no way for the container to know that. Yeah, it has to basically restart that one for the environment starts with the initialization.

12:45
Okay, yeah, great. What about? There was something about, like, dot AWS slash credentials. It would read.

  Speaker 2 12:58
Credentials from that file. What about if we put it in that file, something I don't know anything? Any other media now for containers to?

  Speaker 1 13:08
No, the containers where it kind of Remains the Same. I'll check, uh, that's a good point where? Let's say, if we update on a file, what will happen that? Uh, good question. Whether it will create it dynamically or not. I'll check that, yeah, product change for you. But probably a minor product change, but then again, it will raise a risk of like having Secrets written on disk.

13:37
That's true. That's true. Yeah, that's, uh, I mean, if you consent that as a risk, then yes, I've read that somewhere, but.

  Speaker 2 13:47
Um,

  Speaker 1 13:48
Okay, okay, so it's actually not a product feature update. And because we use Docker and Docker is not ours, so to speak, and the

  Speaker 2 13:58
Way it kind of operates is how it will operate. Um, it's not in our control. Yeah, so. I will check if Portman has something like Secrets or something because. Docker has its own, like Docker Secrets. There's alternative condition over there, which I don't want to bore you to death with.

  Speaker 4 14:22
Docker has a specific feature called Docker swarm.

  Speaker 2 14:28
Yeah, and that's only when she the docker Secrets will come into the picture. So, something like kubernetes Secrets so? Because it's equivalent only in Docker's form and not in the normal darker.

  Speaker 1 14:41
So an effect in any case and see if that will do the trick for us.

14:50
Uh, even in the case of. Um, let's say, and to me, it almost sounds like. Or will be forced over the GSM path, unless can you actually host a VM.

15:07
Get this, you're running over there.

  Speaker 2 15:18
Is some sort of a strategy decision that is pending. Okay, are we going to go to AWS or not? We have provisioned everything in AWS, but will we be? Will that gate be opened for us to go in or not? Is still a pending question? Uh, we also wanted to move all of our Gap implementation.

15:37
Our internal Cloud implementations over to AWS, but that's gated by a decision somewhere I don't know.

  Speaker 1 15:45
Nicey, yeah, uh, duly noted. Do you know that's okay? Uh, seems like I'm going to have an option. So, yeah, I mean, that's, that's okay. I mean, it is what it is at the end of the day. Uh, if then, almost, it almost feels like storing secrets locally is a security risk, which I would relentlessly agree rightfully so.

16:09
Um, I

  Speaker 2 16:10
Can check if we can. I can check with kailash if that can be a possible, you know, temporary workaround or an exception pursue an exception?

  Speaker 5 16:19
I'll definitely keep that enough. I explored that option also, like having a local Secret store, one five on list, great local file secrets.

  Speaker 2 16:30
But that looks like the only way for now without a problem. Well, no, either way, even if it's local, you know, file based secret store, then you still have to make some modifications to HDR, right? No, no, no? Why do you say that? Um. I thought it was only SDR was only able to read from AWS, assuming the AWS role.

16:56
No, no, so I can share that document on the dapper document.

  Speaker 1 17:02
Uh, let

  Speaker 4 17:03
Me quickly check, not check, but give me the link so we can definitely from local file store. That's not a problem. Okay, then. I'll also that's a feasible. Technically feasible option, but allowable or not. I'll check here.

  Speaker 1 17:17
Yeah, uh, so let me actually begin with the least effort or no effort. In fact? Correct. You can actually Google this as well. Like Dapper, that is the APR secret stores. Uh, is what you can check, um, and it'll give you a list of all the secret stores that are supported.

17:41
So, we use this component. Contact tapper the APR, right? And whatever it can. It has binding to. We can upload it for sure. Okay, uh, that's not a problem at all.

18:01
Interesting. It has kubernetes.

18:06
Going up on a tangent? We can. SDR can access into k3s, and then they keep it a secret store. But yeah, I

  Speaker 3 18:14
Was like your documentation. Also, you can deploy the SDR on kubernetes, but I think all that was talking about was, and like, AWS, kubernetes, not on-prem VSI kubernetes.

  Speaker 1 18:26
Yeah, Carlosest, uh, that's true. Ideally, all cubators are fine. I mean, even k3s is okay, but again, the problem is that that becomes the problem. Um, and they purposely don't want people to get involved in KPS. And cared for some something. Usually, there are teams who own that deployment or manage that deployment or the best practices around it.

18:57
I I for now. I mean, it came as a. Uh, a part of security, and hence we were supporting it. And in fact, we didn't even touch KBS purposefully. I mean, we were building on top of it, uh, like cargo was the main. Um, piece, which we were touching equation itself.

19:17
Don't have a lot of experts even in Atlanta. I'll be honest over there, and even if we have, they are completely involved in the tenant management. All the normal platform, which runs on kubernetes. Yeah, it won't be the right call because it's not a product offering per se so.

19:34
There won't be people supporting kps from RN, but yeah, if you have people inside who can support them, great. But I don't think that should be the right part for us. Okay.

19:48
What are the options? What are we going to do next?

  Speaker 3 19:54
Correct. So, in order to summarize, that's that. There's one last Point, um, because I. Against coming through that DSM document with you here.

  Speaker 1 20:02
Uh, given all the challenges which we have a cloud access and whatnot.

  Speaker 3 20:07
Okay,

  Speaker 1 20:07
I am actually inclined to think again. This is not as a product enhancement, but Can you have that cron job, which basically does? Corbros, authentication, and what not. I can try to work, work out a solution where I can tell you that, hey, I, I write

  Speaker 4 20:26
That GSM is basically how she got some modified hashing default as a service. Yes.

  Speaker 1 20:33
Yeah, so that's this Dapper, or in other words, SDR does support hashicon, so it's just the authentication, which I believe is the main problem modified. So

  Speaker 3 20:45
For us, yeah,

  Speaker 1 20:45
Yeah, correct. So that's not something which we can invite, but if let's say another con job of thoughts exists for a program exists real quickly, which allows access. Tom dapper, too, that has he called that could be a potential method is what I'm thinking. Which kind of unblocks us.

21:04
I mean, it will help you comply with that. Exactly what? Uh, it will help with the authentication piece and provide a it will be like a proxy server. In technical terms, which will rule the traffic from the docker. Component, which is running

  Speaker 3 21:24
In. We are the program or the proxy server to your ESM. Okay, at the end, okay?

21:37
I see. Sure I, I will check the feasibility of that and see if we can provide that tooling. The Quran job, yeah?

  Speaker 1 21:50
And I don't want to kind of leave you hanging for you to figure it out. I mean, I will definitely help out to see because the upper is new to you, uh Jason is new to us. Yeah. We both have to come meet somewhere in the middle. I was happy to work as long as we can.

22:09
Uh, collaborate, effectively, right, let's have. Let's work together here.

  Speaker 3 22:14
So, yeah, there's somebody that you were asking sorry, right? I mean, the ideal solution? I believe the first, most ideal solution is.

  Speaker 1 22:22
Having SDR on cloud, but that may be depending on some decision. Yeah, the second is. The local file bar, according to me. Yeah, I will check whether tokens can read it dynamically. That is, if something updates that file, can

  Speaker 3 22:40
The

  Speaker 1 22:40
Container read updated value or not? That's

  Speaker 3 22:42
Right, the second leg of tag. The aspect would be for you to check with Kellage or someone that would be permitted. Okay, the security to have secrets, uh, in a local fine store. Yeah.

  Speaker 4 23:00
I believe is just that. He is something that I mentioned last that you have to put.

  Speaker 1 23:06
A proxy component which we create, which helps route to traffic from SDR via proxy to the GSM server. Okay. All right, okay?

  Speaker 3 23:22
Sure, we'll take it from there. Then, I'll I'll send some notes as well. You can send if you want to. I can send as well, and uh, we will have to get some directions from kailash, I think next, because there's multiple things that we could possibly do. Everything has its own if each one of them has their own effort and timeline.

23:43
Which one does he want us to pursue? Is will have will help us like be efficient at it here.

  Speaker 1 23:51
Sounds good, sir! Thank you so much! Okay, thank you Namas. Let me 7 to me. Here come some of the most plausible path. The amount of challenges that we will have with others. But yeah, even for that, how much effort? Would the teams be involved in something which?

24:21
Thank you. All right. Take care, okay?