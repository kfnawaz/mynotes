Summarize. 

  Speaker 1 00:00
Or store. Why is the foundation and I'll go with storage and within that secrets will change? So you were mentioning this before, uh, so secret. One secret will have one key. Is that correct? Uh, yes,

  Speaker 2 00:13
The so, there is yes. And no, for that question that you had, uh, the GSM allows multiple multiple keys in a secret, but at the same time, it limits. It is, it limits it up to three, and the worst part is it limits. It reserves the first two for itself.

00:35
So, in effect, one secret one, yeah.

  Speaker 3 00:40
Okay, and expect one, one secret, one key, yeah.

  Speaker 1 00:45
Okay, I mean, we can work with it. I mean, in any of those combinations, right? I've been multi multi-key you already saw in the UI multi-key and single key. Yeah, so these both attribute two kind of those. So, if it is one secret, one key we'll go with a single key.

01:01
Um, option option. If it were the other, one one is to end combination. Then, multi-key is something which would have happened. But yeah, this makes sense. Let's stick with the single key approach. Aligned over there. Okay, that was the overall question in general, right? In your

  Speaker 2 01:18
Multiple questions, okay, we just need to reorganize this and have separate keys. That's it, right?

  Speaker 1 01:26
Correct, correct. That's the only thing that is actually blocking us. But for snowflake, once again, uh, if you want, you can actually put in the raw value, but? Uh, I don't know. You will have that. Yeah, window as well, right?

  Speaker 2 01:42
Yeah, I'll have to life, get it, and then put it. Put the right value, but we get we'll get there if we have to, uh. I'm trying to avoid that. Definitely. Maybe

  Speaker 1 01:53
We can work with data bricks, right? Data bricks doesn't have that changing thing.

  Speaker 2 01:57
So, that's a one-time load. So, directions that needs that refresher?

  Speaker 1 02:05
Yeah, correct. So that is correct for data breaks. What are you supplying to Atlanta or what is in the SQL store? It's only this certificate and key, uh, I can show you in in the current setup as well. Right. The current setup won't work. That is why, uh, the whole question?

02:28
Um. Or that's why I started asking, ask question because? Correct. So similar, we'll have to do something similar. Tus Snowflake like, you are giving us the exact. Credential. Uh, we'll need something like that. Yeah, I'm going to. Correct so? Our option right now is precisely even the certificate, and the key is something which I believe you use to.

02:58
Uh, authenticate against ID anywhere.

03:03
And then get the final value, so that final value is something which. You want? Give to Atlanta. Uh, at the end of the day, but. And the final value. Is that the packed open?

  Speaker 2 03:27
No, yeah, correct. It's secure agent and. The connector code itself has.

  Speaker 1 03:33
We are kind of custom build that code, right? But that was the whole point offset problem. Um, okay, okay, that data breach connector was custom built to.

  Speaker 2 03:44
Use the certain key. Current derive that, okay, you, and you don't want to have that logic and probably the new app does not get that logic. So, what you finally want? That's why I was drawing battles with snowflake because you're literally giving us the only piece that is required.

  Speaker 1 04:03
Um, and over there. Yeah, no, I had. I did not recognize the clue that the database was built separately. Uh, what? I've been in the past, so, okay, yeah, that's what I was doing fine. Let's.

  Speaker 2 04:18
Mm, yeah, let me get this store organized first. Uh, and then we will try to run it with the store value for snowflake and worst case scenario. We'll see if the secret can be used. Correct, I think what, uh? We are doing is definitely pack token. That is the current configuration in action.

  Speaker 1 04:41
Like, you're just entering the keyword token, of course, and token is precisely the keyword or the environment variable. Uh, where the back token finally gets stored so? Yeah, that's the, uh, it should be a very small logic, uh, of course, to authenticate with idea anywhere. Particular because all the URLs and everything is given in the configuration right now.

05:06
In today's Dave within security, okay? I don't think it needs any refresh or anything. It's just. Um, a static tokens at the end of the day. So, one time one was

  Speaker 2 05:24
Good. So, in effect, what you're saying is. Data breaks should also follow the same similar pattern as snowflake. From a secret handling perspective. Um, and then that may also end up being a refresher.

05:43
The only difference is the snowflake is an auth token.

  Speaker 1 05:49
And that is a refresher token. Like, I mean, it has an expiry Associated to it, whereas data breaks is a pad token personal access token, so ideally it shouldn't have an expiry readily speaker, so you shouldn't need a refresher. Is my then? That is how the assessment best, guess, right, okay, educated assessment?

06:13
Yeah, then, that means the databricks configuration should be. Oh yeah, that's why this. This pack over here. Okay, okay, the token is the keyword or the environment variable, which in today's code secure agent kind of provides the value in. And if you scroll over here, and these are the.

06:37
Yeah, so these are the configuration items like. Are you anywhere in the URL?

06:48
Get that back token.

  Speaker 2 06:50
Yes,

  Speaker 1 06:50
Yeah, precisely.

  Speaker 2 06:51
So,

  Speaker 1 06:53
It's

  Speaker 2 06:53
Probably one time, then long lived, right? So, exactly, it is. It should be longer as far as I know it is, I believe, more than a year, but? Uh, I don't think it even expires as far as I remember.

  Speaker 1 07:07
All right, so then should I configure

  Speaker 2 07:09
It? Like, obtain this pack token one time, and then save it in as a single secret.

  Speaker 1 07:17
Instead of trying to follow the

  Speaker 2 07:19
Yeah, instead of trying to follow the snowflake through snowflake, no pressure. Okay, yeah.

  Speaker 1 07:24
I am for, to begin with, I would say, yes, uh, but I don't know if there are any standards associated, and it's in jpms for that. Good question. Yes, something

  Speaker 2 07:39
To think about. Okay, let me put one, some thought to it, and see if I want to go ask around about it. Or set up the refresher pattern.

  Speaker 1 08:06
I haven't seen the pack change. Yeah, yeah, I also thinks, oh, I, I wouldn't know because? What's happening in Soviet? Okay, yeah, most likely it might not have changed, but yeah.

  Speaker 2 08:17
So it could be one time, or it could be a very long lived or low refresh rate, I think yeah. The only thing about only reason. I talk about refresh rate for this. Data breaks is like, for the data breaks is, um. I don't know if I'm. If I am supposed to get that token and then see it and put it inside the secret.

08:40
Okay, okay, I'll find it. Okay, sir, I think for now. I think this is our update kind of call if we can. Uh, gather again well and good. I'm happy together again after I'm I have reorganized this R. I will communicate back to you how the new store organization looks like you can on your site.

09:01
Also, go ahead and start, you know. Test this yourself. And just let us know when it starts working right or ask any questions if I can do anything more. We don't have to, I think the way you did yesterday, right? Try to do few things asked bunch of questions.

09:16
That's probably more efficient instead of waiting and trying to do this in 30 minutes. Yeah. It's

  Speaker 1 09:21
Break.

  Speaker 2 09:51
Is basically educating me as well. So, whatever, I feel okay. Oh, this might go wrong. I try to include that in the emails to you.

  Speaker 1 10:00
I, I cannot say that day. I'm intelligent enough to cool talk as well, so I'll be honest. I'm

  Speaker 2 10:10
Sure it's an idea. Okay, yeah,

  Speaker 1 10:12
All right, sir. Let's keep it efficient. You can probably have your time back, and I'll as soon as I'm done here. I'll maybe back. Works okay. Thank you so much. Take care. Have a good night. Thank you. Thank you. Thank you very much.

--------------

## **Meeting Summary – GSM Secret Store, Snowflake & Databricks**

### **Objective**

Align on GSM secret storage structure and credential handling for Snowflake and Databricks connectors in Atlan.

### **Key Findings & Decisions**

- **GSM Limitation:** Although GSM supports three keys per secret, two are reserved internally, effectively allowing only one usable key per secret.
- **Agreed Approach:** Use **single-key secrets** instead of multi-key configuration. Existing secrets must be reorganized accordingly.
- **Snowflake:** Requires an authentication token with an expiry, making token refresh necessary. Testing will proceed once the secret store is reorganized.
- **Databricks:** Currently uses custom connector logic to retrieve a Personal Access Token (PAT) through IDAnywhere using certificates and keys.
- **Databricks Proposed Approach:** Retrieve the PAT and store it directly as a single secret, similar to Snowflake, avoiding custom token-retrieval logic.
- **Token Lifecycle:** Databricks PAT appears long-lived, potentially requiring infrequent or no refresh. Actual expiry and organizational requirements remain unconfirmed.

### **Blockers / Open Questions**

- GSM secret store must be reorganized before testing can proceed.
- Confirm whether JPMorgan security standards permit storing Databricks PATs directly or require automated refresh.
- Determine whether existing Databricks token-retrieval logic needs to be retained.

### **Action Items**

1. **Secret Store Team:** Reorganize GSM secrets into the agreed single-key structure.
2. **Snowflake:** Test connector authentication using the reorganized secrets.
3. **Databricks:** Validate PAT expiry, storage requirements, and refresh expectations.
4. **Connector Team:** Test configurations independently and communicate issues asynchronously.
5. **Both Teams:** Reconnect after store reorganization if further troubleshooting is needed.

### **Conclusion**

The teams aligned on a **single-key secret storage approach**. Snowflake testing is the immediate priority, while Databricks credential handling requires confirmation of token lifecycle and security standards. Asynchronous testing and communication will be preferred over waiting for scheduled troubleshooting calls.