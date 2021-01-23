--- 
title: "Adding Chat widget for RASA bot"
description: "Tips and tricks collated from various sources"
layout: post
toc: true
comments: true
hide: false
search_exclude: true
categories: [RASA]
author: Vansh Kapil
metadata_key1: metadata_value1
metadata_key2: metadata_value2
---

```jsx
Code chat widget https://github.com/botfront/rasa-webchat
```

Here is the basic chat widget code to be placed in <Body> </Body> tags


```jsx
<div id="webchat"></div>
<script src="https://cdn.jsdelivr.net/npm/rasa-webchat@0.11.5/lib/index.min.js"></script>
<!--// you can add a version tag if you need, e.g for version 0.11.5 https://cdn.jsdelivr.net/npm/rasa-webchat@0.11.5/lib/index.min.js-->
<script>
  window.onload = WebChat.open; 
  WebChat.default.init({
    selector: "#webchat",
    initPayload: "main menu",
    customData: {"language": "en"}, // arbitrary custom data. Stay minimal as this will be added to the socket
    socketUrl: "http://localhost:5005/",
    socketPath: "/socket.io/",
    title: "This is Title",
    subtitle: "AI Assistant",
    inputTextFieldHint: "Type a message",
    embedded: false,
    showFullScreenButton: true,
    showMessageDate: false,
    hideWhenNotConnected: false,
    displayUnreadCount: true,
    profileAvatar: "https://static.wixstatic.com/media/7309b2_23f0b6a0511741648ca7b21acfb1fa6d~mv2.png",

    params: {"storage": "session"} // can be set to "local"  or "session". details in storage section.
  })
</script>
```

**socketUrl** in the above code is the url of your bot; 

### Running webchat on [localhost](http://localhost) with RASA X

in case of bot (with RASA X) running on local machine, under default settings. 

```jsx
socketUrl: "http://localhost:5005/"
```

Step 1 - rasa run actions

step 2- rasa x

### Running webchat on [localhost](http://localhost) without RASA X

Step 1 - rasa run action

Step2 - rasa run -m models --enable-api --cors "*" --debug

Step3 - update endpoints

![]({{site.baseurl}}/images/Adding-Chat-widget-for-RASA-bot/Untitled.png "image")

step 4 - update URL [http://localhost:5005](http://localhost:5005/) in html ( same port on which server is running)

### In case bot is on server, for example zuzu

```jsx
socketUrl: "https://zuzu.sundaybots.com"
```

### Directing chatbot according to the page

```jsx
initPayload: window.location.pathname,
```

When a page loads , the above parameter will send the pathname of the current page. 

```jsx
initPayload: window.location.href,
```

This passes the entire URL 

Detecting the current page, bot can initiate an intent.

If the bot is already initialized, add the below code after **WebChat.default.init()** 

```jsx
var locn = ['/check_url{"page_url":"',window.location.href,'"}'].join('')
WebChat.send(locn)
```

This will send the current URL to RASA without printing the text on chat.

More Details [https://github.com/botfront/rasa-webchat#api](https://github.com/botfront/rasa-webchat#api)

### Opening chat widget on page load

```jsx
window.onload = WebChat.open; 
```

Adding this command before widget function. 

code looks like

```jsx
<script>
  window.onload = WebChat.open;
  WebChat.default.init({
    selector: "#webchat",

```

### Creating a hyperlink in chat widget

```jsx
[Sundaybots](https://www.sundaybots.com/)

[TITLE](URL)
```

### Carousel in Chat widget

```jsx
	test_carousel = {
            "type": "template",
            "payload": {
                "template_type": "generic",
                "elements": [{
                    "title": "Golf",
                    "subtitle": "Super Slow Sport",
                    "image_url": "https://static01.nyt.com/images/2020/07/23/sports/23golf-wie-big/23golf-wie-big-videoSixteenByNineJumbo1600.jpg",
                    "buttons": [{
                        "title": "Golf Link name",
                        "url": "https://www.golfchannel.com/",
                        "type": "web_url"
                    },
                        {
                            "title": "Golf postback name",
                            "type": "postback",
                            "payload": "/greet"
                        }
                    ]
                },
                    {
                        "title": "Cricket",
                        "subtitle": "Best Game in Town",
                        "image_url": "https://nnimgt-a.akamaihd.net/transform/v1/crop/frm/silverstone-feed-data/67c54b2a-b43b-4cbd-9a83-cb5208d9a5b5.jpg/r0_0_800_600_w1200_h678_fmax.jpg",
                        "buttons": [{
                            "title": "Cricket Link name",
                            "url": "https://youtu.be/_imizBMHN0w",
                            "type": "web_url"
                        },
                            {
                                "title": "Cricket postback name",
                                "type": "postback",
                                "payload": "/greet"
                            }
                        ]
                    }
                ]
            }
        }
        dispatcher.utter_message(attachment=test_carousel)
```

### Color Font and widget styling

```jsx
<style>
		div.rw-message-text
	{
		font-size: 12px;
	}
	div.rw-header.rw-with-subtitle{
		background-color: rebeccapurple;
	}
	.rw-open-launcher{
		background: url("https://static.wixstatic.com/media/7309b2_23f0b6a0511741648ca7b21acfb1fa6d~mv2.png") no-repeat;
	}
	.rw-conversation-container .rw-response{
		background-color: light;
	}

	.rw-conversation-container .rw-client{
	    background-color: rebeccapurple;

	}
	.rw-widget-container .rw-launcher{
	    background-color: rebeccapurple;
	}

	</style>
```

**To change quick replies and properties**

![]({{site.baseurl}}/images/Adding-Chat-widget-for-RASA-bot/Untitled1.png "image")

```jsx
.rw-conversation-container .rw-replies {
        display:inline-block;
    }
.rw-conversation-container .rw-reply{
        display:block;
        text-align: center;
        color: green;
        border-color: green;
				font-size: 14;
        font-weight: bold;

    }
```

**Button Background color**
![]({{site.baseurl}}/images/Adding-Chat-widget-for-RASA-bot/Untitled2.png "image")


```jsx
.rw-widget-container .rw-launcher{
	    background-color: green;
	}
```

**Header Banner**

![]({{site.baseurl}}/images/Adding-Chat-widget-for-RASA-bot/Untitled3.png "image")

```jsx
div.rw-header.rw-with-subtitle{
		background-color: rebeccapurple;

	}
```

**Background bubble on the bot reply**

(light grey in this example)

![]({{site.baseurl}}/images/Adding-Chat-widget-for-RASA-bot/Untitled4.png "image")

```jsx
.rw-conversation-container .rw-response{
		background-color: light;
	}
```

**Background bubble of user message**

(red in this example)

![]({{site.baseurl}}/images/Adding-Chat-widget-for-RASA-bot/Untitled5.png "image")

```jsx
.rw-conversation-container .rw-client{
	    background-color: red;

	}
```

### Adding Video to response

sending video attachment from domain.yml

```jsx
utter_greet:
  - text: "Hey! How are you?"
    attachment: { "type":"video", "payload":{ "src": "https://youtube.com/embed/9C1Km6xfdMA" } }
```

sending video attachment from [actions.py](http://actions.py/):

```jsx
def run(self, dispatcher: CollectingDispatcher,
            tracker: Tracker,
            domain: Dict[Text, Any]) -> List[Dict[Text, Any]]:
        msg={ "type":"video", "payload":{ "title":"Link name", "src": "https://youtube.com/embed/9C1Km6xfdMA" } }
        dispatcher.utter_message(text="Hello World!",attachment=msg)

        return []
```

The link should be in embed mode not anyone other mode. Eg [https://youtube.com/embed/9C1Km6xfdMA](https://youtube.com/embed/9C1Km6xfdMA)   

More details, original source

[Click Here](https://forum.rasa.com/t/displaying-video-in-the-rasa-webchat/26931) 

for telegram, follow custom action code works

```jsx
dispatcher.utter_message(text=txt, attachment="https://youtu.be/PN3S-e6I4IQ")
```

Some more info

```jsx
https://forum.rasa.com/t/rasa-web-chat-customizing-buttons-colors-alligning-etc/24716/28
```