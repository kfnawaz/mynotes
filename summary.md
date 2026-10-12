Summarize this as an action plan with the steps to go through. 

  Speaker 1 00:00
Sorry, how does it work? Sorry, what is the proposed solution? So, so you create a saml enabled role right inside an AWS account? Okay, and once you create a family enabled, what happens is that role will have a corresponding ad group created. Okay, so that you can? You can also end on the vsi side.

00:20
You will have an FID trying a functional, functional ID that functional ID gets added to an ad group. Okay. So your vsi application will basically use that function ID to assume. To log into AWS account, it will assume that I am rolled the family animal IM rule, yeah. And that's how you will get inside the AWS account.

00:43
On other hand, when you provision the saml animal IEM role, you're going to give it a permission to do the get secret value from secret resource. So, that is, how you will be able to establish the connectivity from your vsi to. Um. To AWS account. Okay. So, here was the description of the ticket that we created, which had proposed solution pretty much parallel to what you are just saying here.

01:13
For your visibility. I'm gonna copy paste this. Not sure if you're able to see that ticket or not. Details or not, but the the request details are right here. Okay, so this is the ticket you submitted correct, and uh. You're saying somebody provided?

  Speaker 2 01:32
Solution solution. Oh yeah, a possible solutions, let's say, right? Yeah, Federated identity.

01:44
If I have a principal broker? Yeah, I think they will go to the code. Go through the same thing, right? Your associate vsi functional, right? I'm just reading with the roles of the broker assumes. Yeah, it's the same thing I'm I'm describing here and centered. Okay, yes. So, how do I get this done?

  Speaker 1 02:03
Now that we know, okay, this is this proposed one is likely the very similar thing pattern that you are also. Ah, prescribing for your cpmc users. How do I go about getting this tensor? Okay, so I, I don't have. The solution because my, I'll show you how, how you can get this done.

02:23
I'll try to explain best of what I can. Okay, we don't own this Tech primer. We don't know, publish the solution because our team, we are platforming for cigarettes manager, not for a role, but I'll let me share my screen. I'll share couple links and then engines. Sure, an engineer from your team will have to.

02:45
Get their hand sanity and sound run through this plan, right? Yeah, so basically, you will, uh, here. This Tech primary is to set up a secret manager service, right? So how you gonna create a secret? I am assuming you guys probably have your own AWS account where you are provisioning a secret resource and all that.

03:02
Yes, so this? Okay, so you will be following this as a part of it. We are here. Could you bring this link? I believe we've already followed this same Tech primer, right? Yeah, yeah, this will. This will only get you set up with your secret manager service and a robot, and that role will have a permission to do the get secret value.

03:21
Okay, what you need is when okay here. When you are provisioning the role right now, this saml animal is false. You will have to make this true. You will have to enable you will have to. You'll have to create a saml enabled role. Nice, right, and then there will be no agreement service, because who's gonna assume the Samuel animal role is your F5?

03:40
From the on-prem. From your vsi box, there is no AWS service assuming this role. So, uh, you will cite this to remove this part. And then again, you can reference the role module and I'll ping you all these link. Let me just quickly walk you through what you're gonna do.

03:55
So, if you? Uh, if you go to terraform module. There is a terraform module for role, and they will have. But let me bring it up. It's easier that way they will have if you for any terraform registry. If you go to the pay folder. They will have an example of what all they support, so you will basically.

04:12
You can look at this and find. A role module block where that, yeah. So, this is a sample enabled role right where they are setting it to true. So, you know what parameters to pass and whatnot once you create this role? You want to make sure you give it all the permissions to do the get secret value so that you can reference here.

04:34
Basically, this whole policy update or module I'm giving read access equal to true, and this is the role name, which will be your simulator once those things go through. Will go to get at last data log.

04:53
And you will set up manage access to Cloud.

05:00
Um, I'm just gonna. I'm able to find something or not, but basically you will select your AWS account right. Like, for your applications, your AWS account and then? The group to manage this is the Ed group, which would have been created for your saml role. So, for example here, if you see, I have an app custom saml roll.

05:27
So, whatever saml role you can you create ad group gets ad group gets created. You select the ID group, and here, you're gonna add your, uh, not the Sid. We're gonna add the functional ID, so FID Source application. Basically your vsi application. If it is not on the same seal, then this will.

05:47
This

  Speaker 2 05:47
Will be different, but let's assume it's on the same same application. Okay, so which database account belongs? You're going to select your FID. So

  Speaker 1 05:54
These are

  Speaker 2 05:54
The fides I have under my EPV tools application. I'm going to select one of them right, and they will do bunch of validation and whatnot. Here. And yeah, that's sure. So, that provides some justification. Yeah, to create the ticket, I guess the request for onboarding?

  Speaker 1 06:13
Correct, but can we okay, anything else, like, Beyond this, I wanted? No, I wanted to verify that, uh, the I the IDs, right? Which FID are we talking about the functional frid that we use to log in to the vsi, or it should be another F new FID, or that I I'm not a bit clear on it, it could be it.

06:33
It would probably be a new F5. If you want to create a dedicated FID, which will be able to log, go inside the AWS account and assume. Then, that's where I would start. Yeah, you? Then you can request. Yeah, if you, if you have an FID? I think usually AO.

06:53
Can make a call on that if they want to go ahead and? Could you use the existing s5d? Yeah, I am the application owner, so I wanted to like identify which FID would be the right one to, you know, start with on here. If I can share my screen, I'll show you, which, what environment our environment looks like and what is the?

07:12
Fids that we have, and so forth. Yeah, uh.

  Speaker 2 07:16
Yeah, you can. You can do that. I'm not. I'm not sure if I'll be. No, I'm in the right person to answer that. I mean, the better. Your answer will be better than. Hopefully better than what we could think of so. Let me share my screen.

  Speaker 1 07:38
You can see my screen. Yeah. So?

07:49
And I'll open fids also.

08:16
That, that is our, uh? Account system account. So that using that account is what our application runs, the okay SDR, the orchestrator that we're talking about. That's what it runs as.

08:41
Just a second.

08:50
So, Atlant edit? Uh, how do I find what? What should it be the atlan edit or? Uh, go AWS console. Also, I'll go log in there.

09:10
I mean, alternate it right there. All right, Atlanta. And the accounts that we have.

09:38
So, we have this, uh, whole bunch of accounts. But this is our Dev one that we're going to use.

09:52
I want it to take you to the Secrets manager. What we have done so far?

  Speaker 2 09:58
Oh, that's fine. I don't need to know that now. I, the other, will probably run out of time. Oh, you have you have other, uh, yes, also. Yeah, I, I have another call after this, so?

  Speaker 1 10:11
Oh, after this, okay, yeah, sure, yeah. So, these are the secrets that we are talking about one, two, three.

10:22
Something is showing the second one, let's say now.

  Speaker 3 10:28
Woman, let's look. Look like when we have some more enabled secret created place,

  Speaker 2 10:36
It will. It will not look any different. See secret is one thing the animal animal you're creating a salmon. That role is gonna have permission to read secrets. Okay, yes, so there's nothing, uh?

10:54
Yeah, there's nothing else you are doing different with the secrets. We created the role as well, right, uh, right, right?

  Speaker 3 11:05
Which is here. And. As they are this row.

  Speaker 1 11:25
In Guru trust relationship.

  Speaker 2 11:29
Yes, so here, if you see the service, this is a loud is for easy to write. Only ec2 can use it. Now, you want to use it from on-prem, so it's that's why you're going to create a sample enabled role, and then you will have a Federated trust policy as the Ida something, because basically this raw role.

11:47
The authenticated against, uh, idea. I did Idaho identity, so the this section, the principle section will look different when you provision a similar enabled rule basically. Okay, so it has there ever been a scenario like this. Like, somebody is an example that we can see. Yeah, for vsi, yes.

12:14
Uh, not for vsi, not on top of my mind. I can. Think there is a team who is trying to connect from vsi to S3 bucket, so they will probably have. Uh, yeah, I, I can connect you to somebody. They are again. They are not a support team. So, yeah, yeah.

  Speaker 1 12:39
Okay, yeah, they probably have something similar challenges

  Speaker 2 12:43
Correct. They are going through this so, but like I said, it will be there is. There might be a documented solution somewhere which my team doesn't know, so I don't know, but the terraform links I gave you. You can. Somebody an engineer will have to spend some time on like there is no template where you'll be able to copy, paste it, and make everything work in one shot.

13:04
Where did you ping the links, sir? Uh, I can bring it to you on the same chat where you added me. Yeah, yeah.

  Speaker 1 13:21
Yes, true trust. Yeah, that chat. Yes, okay.

13:29
But also the the link to?

  Speaker 3 13:35
Uh, yeah, it's Atlas Casino. I'm not sure the big bucket.

  Speaker 2 13:42
Yes, bitbucket. You probably won't have access being an lob team. I'll give you the link where they publish the same.

13:53
Examples for every every any module. You can basically go through the examples, its own terraform registry. You will have access there actually.

  Speaker 1 14:12
You won't have access to code where all those things get published into TFE sorry Atlas module registry. You will have access there because you probably have a workspace created for your AWS account so. Yeah. Okay, so there will be some importance. I'll try to connect you to. An engineer from the other team who is, uh?

  Speaker 2 15:01
Now, what's the other thing? Yeah, we are the team. Okay, yeah, sorry, S3. Other team. They're trying to connect to S3 buckets, it seems. Yeah, from BSN.

  Speaker 1 15:19
We are trying to connect to secret manager. So they will require very similar rules, I believe. Permissions.

15:33
Anything else Emmanuel? We can probably learn from month and here. Uh, I'll

  Speaker 3 15:41
Go through the the the go Engineers link that is and over there and then look at the tech primer for the thermal enabled ro.

  Speaker 2 15:53
Yeah, Emmanuel. The best thing is, this is not. This is probably going to take less than a couple hours. We've already provisioned the secret. You just add a semal enabled role, which is like literally four lines in there provision a role and the permissions right and. Just use whatever fight you might have, or to create a new FID anything in lower environment.

16:12
Things should be fast. Nawaz is your area. You'll approve whatever and try to establish that connectivity. You are able to assume the role or not. All I'm saying is. Run through this. Just do it practically nice, and you'll hit our error or a block, and then you. Aske lelema.

16:31
Try to figure it out is the best.

  Speaker 3 16:32
Yeah, yeah. So before we leave, I have one quick question because let's say I have commissioned the role you went to book Atlas catalog and then you were able to click on feed and then you're able to select a certain role because the role that I created in Native U.S appears when I go to Manchester.

16:52
So, it appears automatically correct. Correct. It will create an ad group, and that's the. That's why you are. So it create gets created in IA AWS account. At the same time, ad group gets created with the same name. Okay, okay, that makes sense, so that's basically like a breed.

  Speaker 2 17:12
Roll between your on-prem and AWS. The only cavities in PSI, because if you have an FID. I, I don't know what type of FID you will need. I think you'll need an Idaho based FID, because if you are in BSI. You can. You can't have a password for that fid because you can't Vault it.

17:31
You'll have to watch the FID somewhere otherwise. I think you'll get a break, so that's. There might be some work needed, but I'm sure you guys have app in the vs will be able to figure that out. Uh.

  Speaker 1 17:48
Okay, thank you so much. I appreciate it. Thank you.

# Action Plan: VSI Access to AWS Secrets Manager

## Objective

Enable the VSI-hosted SDR/orchestrator to retrieve secrets from AWS Secrets Manager using a SAML-enabled IAM role and an approved functional ID (FID).

The existing secrets remain unchanged. The current IAM role trusts EC2; VSI access requires a federated trust configuration.

## Steps

1. **Gather the documentation**
   - Obtain the Secrets Manager tech primer.
   - Review the SAML-enabled IAM role examples in the Atlas/Terraform module registry.
   - Obtain the Manage Access to Cloud link.
   - **Owner:** Emmanuel / engineering team.

2. **Select the development test scope**
   - Identify the development AWS account and VSI application.
   - Select one existing secret for testing.
   - Record the AWS account ID and secret ARN.
   - **Owner:** Engineering team.

3. **Confirm the appropriate FID**
   - Decide whether to reuse the SDR/orchestrator’s existing FID or create a dedicated FID.
   - Confirm the FID type supports federation and unattended execution from VSI/BSI.
   - Confirm any approved credential-vaulting requirements.
   - **Owner:** Nawaz, with identity support.

4. **Provision the SAML-enabled IAM role**
   - Follow the approved Terraform role-module example.
   - Set SAML enablement to `true`.
   - Configure federated trust instead of EC2 service trust.
   - Deploy the role in the development AWS account.
   - **Owner:** Emmanuel / engineering team.

5. **Grant secret-read permissions**
   - Grant `secretsmanager:GetSecretValue` access to the required secrets.
   - Verify the policy references the correct role and secret resources.
   - **Owner:** Engineering team.

6. **Onboard the FID to the role’s AD group**
   - Confirm the corresponding AD group was created during role provisioning.
   - Open Manage Access to Cloud.
   - Select the AWS account and the role’s AD group.
   - Select the application’s FID.
   - Provide justification and submit the access request.
   - Complete the required application-owner approval.
   - **Owner:** Engineering team submits; Nawaz approves where required.

7. **Configure authentication from VSI**
   - Implement the approved federation/broker authentication flow.
   - Enable the application to authenticate with the FID and obtain temporary AWS credentials for the role.
   - Confirm credential handling and renewal for unattended execution.
   - **Owner:** Engineering team, with identity/federation support.

8. **Test end-to-end access**
   - Authenticate from the development VSI.
   - Assume the SAML-enabled role.
   - Retrieve the selected test secret.
   - Repeat the test through the SDR/orchestrator.
   - Ensure secret values are not exposed in logs.
   - **Owner:** Engineering team.

9. **Resolve blockers and document the implementation**
   - Capture specific authentication, AD membership, trust-policy, or permission errors.
   - Engage the appropriate support team.
   - Follow up on the offered introduction to the team accessing S3 from VSI.
   - Document the working configuration before extending it to other environments.
   - **Owner:** Engineering team.

## Key Open Question

Which FID type and authentication mechanism should the VSI application use to obtain its SAML assertion?

The meeting outlined role provisioning and access onboarding, but did not confirm the application’s authentication flow. Resolve this early with the identity/federation team.

## Immediate Next Actions

- **Emmanuel:** Review the Terraform examples and begin provisioning the development SAML role.
- **Nawaz:** Confirm the FID choice with the identity team and approve the access request when submitted.
- **Engineering team:** Validate authentication and secret retrieval from VSI.

## Completion Criteria

- The approved FID can authenticate from VSI.
- The application can assume the SAML-enabled IAM role.
- The SDR/orchestrator can retrieve the required secrets.
- Unattended credential renewal works.
- The implementation is documented and repeatable.

## Timing Note

The meeting’s “couple of hours” estimate referred to initial role provisioning, not the complete integration.