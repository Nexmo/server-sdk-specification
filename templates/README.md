Nexmo Client Library for [LANGUAGE]
===================================
[BADGES, BUILD STATUS, ETC]

You can use this [LANGUAGE] client library to add [Nexmo's API](#api-coverage) to your application. To use this, you'll 
need a Nexmo account. Sign up [for free at nexmo.com][signup]. 

 * [Installation](#installation)
 * [Configuration](#configuration)
 * [Usage](#usage)
 * [Examples](#examples)
 * [Coverage](#api-coverage)
 * [Contributing](#contributing) 


Installation
------------

To use the client library you'll need to have [created a Nexmo account][signup]. 

To install the [LANGUAGE] client library using [PACKAGE MANAGER]:

    COMMAND, CONFIGURATION

Alternatively you can [CLONE, DOWNLOAD]

    EXAMPLE CLONE COMMAND


Configuration
-------------
_Use this *or* 'Usage', depending on the language norms._

To configure Nexmo [LANGUAGE] in your application:

Some step 

    EXAMPLE OF THAT STEP
    
Another step

    EXAMPLE OF THAT STEP

Usage
-----
_Use this *or* 'Configuration', depending on the language norms._

To use Nexmo [LANGUAGE] in your application, create a instance of the client library using your Nexmo API credentials. 

    EXAMPLE CODE

Examples
--------
The following examples show how to:
 * [Send a message](#sending-a-message)
 * [Receive a message](#receiving-a-message)
 * [Initiate a call](#initiating-a-call)

### Sending A Message

Use [Nexmo's SMS API][doc_sms] to send an SMS message. 

    //example code showing the client sending an SMS message, and using the response


### Receiving a Message

### Initiating a Call

### [Additional Examples]


API Coverage
------------

* Account
    * [X] Balance (`account.balance`)
    * [ ] Settings (`account.settings`)
    * [ ] Top Up (`account.topUp`)
    * [ ] Update Password (`account.updatePassword`)
    * [ ] Update SMS Webhook Callback (`account.updateSmsWebHookCallback`)
    * [ ] Update Delivery Receipt Webhook Callback (`account.updateDeliveryReceiptWebHookCallback`)
* [ ] Numbers
    * [ ] Search (`number.search`)
    * [ ] Pricing (`account.getPrice`)
    * [ ] Buy (`number.buy`)
    * [ ] Cancel (`number.cancel`)
    * [ ] Update (`number.update`)
* Number Insight
    * [ ] Basic (`numberInsight({level:basic})`)
    * [ ] Standard (`numberInsight({level:standard})`)
    * [ ] Advanced (`numberInsight({level:advanced})`)
    * [ ] Check Webhook Notification Signature (`webhook.validate`)
* Verify
    * [ ] Start (`verify.start`)
    * [ ] Check (`verify.check`)
    * [ ] Search (`verify.search`)
    * [ ] Control (`verify.control`)
* Messaging 
    * [ ] Send (`sms.send`)
    * [ ] Check Delivery Receipt Webhook Signature (`webhook.validate`)
    * [ ] Check Inbound Messages Webhook Signature (`webhook.validate`)
    * [ ] Search 
        * [ ] Message (`sms.search(messageId)`)
        * [ ] Messages (`sms.search({ids:[], date:DATE, to:NUMBER})`)
        * [ ] Rejections (`sms.search({rejected:true, date:DATE, to:NUMBER})`)
    * US Short Codes
        * [ ] Two-Factor Authentication
        * [ ] Event Based Alerts
            * [ ] Sending Alerts
            * [ ] Campaign Subscription Management
* Voice
    * [ ] Outbound Call (`voice.call`)
    * [ ] Text-To-Speech Call (`voice.textToSpeech`)
    * [ ] Text-To-Speech Prompt (`voice.textToSpeechPrompt`)
    * [ ] Check Inbound Call Webhook Signature (`webhook.validate`)

Contributing
------------

[TBD]

License
-------

This library is released under the [MIT License][license]

[create_account]: https://docs.nexmo.com/tools/dashboard#setting-up-your-nexmo-account
[signup]: https://dashboard.nexmo.com/sign-up?utm_source=DEV_REL&utm_medium=github&utm_campaign=[LANGUAGE]-client-library
[doc_sms]: https://docs.nexmo.com/api-ref/sms-api?utm_source=DEV_REL&utm_medium=github&utm_campaign=[LANGUAGE]-client-library
[license]: LICENSE.txt
