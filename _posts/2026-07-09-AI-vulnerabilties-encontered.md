---
title: AI implementation vulnerabitlites encoutered in the wild
description: This blog describes AI implementation issues encountered in the wild as of 2026
author: Dennis
date: 2026-07-09 10:18:00 +0800
categories: [Cybersecurity]
tags: [security, AI security]
pin: true
---


# AI implementation vulnerabitlites encoutered in the wild

Writing this blog in July 2026, it is fascinating to see how numerous companies have integrated AI into their applications in a variety of unique ways. Here, I will discuss a few of the security mishaps I came across in public programs regarding the implementation of these AI features.

##  Unsafe Cross-Origin Communication in AI Briefings, leading to a Hijacked UI
{: .mt-4 .mb-0 }

In this part of the blog I will detail a security issue I came across a famous public program and due to constraints we shall call it `example.com`

While investigating the features of `example.com` I noticed a feature that allowed users to use AI to summarise content eg blogs that were shared and meeting notes. Most important with this was that the AI was in another domain eg `ai.example.com` and the main application was in `app.example.com` if you know anything about browser security you will know that those two are in different domains hence same origin policy will prevent communication.

Upon further investigation I did notice  that the AI would actually receive a post message and send back a postmessage to the main application with the summarized content which would then be rendered in a modal that also allows for “audio summary”

The entry point of the code looked like this. I have redacted the exact function names for privacy purposes
```js
export const Audioframe = (): void => {
	const { audioplayer } = player();

	useEffect(() => {
		const handleMessage = (event: MessageEvent) => {
			if (!isAudioPlayerIframeMessage(event.data)) {
				return;
			}
			Player(event.data.Content, event.data.options); // entry point
		};

		window.addEventListener('message', handleMessage);

		return () => {
			window.removeEventListener('message', handleMessage);
		};
	}, [Player]);
};
```

As you can see the application only checked the format of the message and crucially missed the origin checks. 

To quickly test such scenarios, I opened up `localhost` on my computer and send a post message.
Sample of such code is 

```js
const tgt = window.open("https://app.exampl.com/modal","tgt");

tgt.postMessage({PAYLOAD}, "*");

```

In this case the payload contained a link that rendered on the page importantly the link was  not sanitized hence it was plausible to put a jsuri such when clicked executed xss.

In such cases xss usually leads to account takeovers but the application had correctly scoped cookies which were `http only` hence the next best thing is to look for a request that contained state changing attributes and coerce the application to send it on behalf of the victim. 


Impact: When a victim, however, had to click the share button for the xss to fire which then executes JS.

The mitigation for this was simple:
The application only needed to have proper origin check functionality in their AI integrations to refuse messages from any origin


## Wolf in sheeps clothing

In this part of the blog I will explain an impersonation attack ie sending prompts as the user and coercing the AI to benign actions as sensitive behaviour
I came across an application that implements AI features into the app. This application was unique in that while browsing any part you would just type `“/[command]”` and an AI chat would pop up with the page context already loaded.

While investigating how this works I noticed as well that the model was in another domain and that the page had registered a postmessage listener which would actually just take data from the AI chat panel spawned by the `/{command}` and push it to the model. 

This was no unique missing origin as it wouldve easily been spotted by automation and other hackers as well. It turns out the application had rewritten or extended browser apis and had even renamed the postmessage function to a custom name so I had to dive into the js to decode its functionality



On first look the entry point was clear the code looked like

```js
onMessage(handler: (data: RPCMessage) => void): void {
// …snipped

const handler = (event: MessageEvent<TransportOBJECT>) => this.handler?.(event.data); // vulnerable entry point
            targetWindow.addEventListener('message', handler);
            this.cleanup = () =>
targetWindow.removeEventListener('message', handler);
        }
    }
``` 
A common mistake I encounter is that applications tend to check the format and encoding of the data or do a lot of type casting and fail to check origin, and or have lose implementations of origin checks as we can see from the above code.

Interestingly, the message seemed to listen to a specific command then pass the data to an RPC bridge that would then pass along the data into the AI model,

Some of the commands that were available in the RPC map handler were

```json

{“chat”:”content-ready”,
“Chat”:”content-destroy”,
“Chat”:”sendinlinecommand”,
…snipped
}

```

<!-- tip goes here: In order to find the map with the fucntions one must know how to use dev tools to put break points and call the fucntion from the terminal-->
> In order to find the map with the fucntions one must know how to use dev tools to put break points and call the fucntion from the terminal
{: .prompt-info }

The overview of the source-to-sink flow was multi-step

` BrwoserTransport -> contentBridge -> registersthehandlerFromRPC -> passDataToModel -> modelExecutescommand `

This was found in a public program with lots of researchers but I believe others missed this as one had to plough through alot of large multiple js files. AI did help but not fully autonomous as I had to guide it to where to look.

The impact for this was what the AI had access to. Exfiltrating data was difficult hence actions such as completing certain app actions calling other tools that the model had. To prove impact I had the model delete data that it had access to

Mitigation was simple as the origin check was forgotten once the data format was changed and typecasted. Also the application did note that sensitive actions such as create update delete that the model had should have a guardrail and ask the user for explicit permission before acting.

