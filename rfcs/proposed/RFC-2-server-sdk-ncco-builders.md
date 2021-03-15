# Server SDK NCCO Builders

## Problem

Many developers, both in and outside of the company, struggle with composing correct NCCO actions for the Vonage Voice API. This can be for many reasons, but primarily, it is because each NCCO action has its own specific structural and data requirements that can be hard to remember when building them. The Vonage Voice API does not provide comprehensive error response messaging for invalid NCCOs either, which leaves users in a difficult position.

## Duration

## Current State

Proposed

## Proposers

Ben Greenberg

## Detail

The composition of valid NCCO actions are a required development component in constructing voice communications with the Vonage Voice API. The fine attention to idiosyncratic details that is required to create a correct NCCO is often a stumbling block for customers, both those who are new and those who are experienced. Vonage currently offers an [NCCO Reference Guide](https://developer.vonage.com/voice/voice-api/ncco-reference) on the API Developer Portal, but that guide does not provide real-time support for users developing in their environment. Additionally, the Vonage Voice API itself provides minimal error response support for invalid NCCO actions.

The challenge of composing correct NCCO actions is one of the most frequently raised support issues in the various Vonage support challenges (Slack, Stack Overflow, GitHub Issues, etc.). All of our mainline SDKs fully support the Voice API, and provide to differing levels, support for the composition of NCCOs. This NCCO support though varies widely from one SDK to the next. In some of the SDKs the NCCO actions are treated as pass-through parameters with no validation or assistance, whereas in other SDKs there are more complete validators and builder support mechanisms. 

We can offer a comprehensive NCCO builder in each of the mainline Vonage SDKs that will ease the NCCO action composition burden on our users considerably. These builders will assist users in both creating their NCCO actions, and provide real-time feedback on incorrect construction. 

## Proposed Solution

### Technical Specifications

The NCCO builder MUST be instantiable. By providing distinct instantiations for each NCCO action, it becomes easier to provide specific feedback on each unique action for the developer. For example, there may be multiple `type` parameters throughout the NCCO actions, but the specifications for `type` will be different for each unique NCCO action. The disaggregation and unique instantiation of each action as its own class allows for that fine attention to detail.

```ruby
# Standalone NCCO with two actions

talk = Vonage::Voice::Ncco.talk(text: 'Hello World!')
input = Vonage::Voice::Ncco.input(type: ['dtmf'], dtmf: { bargeIn: true })
ncco = Vonage::Voice::Ncco.build(talk, input)

# => [{:action=>"talk", :text=>"Hello World!"}, {:action=>"input", :type=>["dtmf"], :dtmf=>{:bargeIn=>true}}]
```

Each NCCO action MUST be able to be accessed via its own method name accessed on the NCCO class. For example, the `connect` action would be accessed as `Vonage::Voice::Ncco.connect` and the `notify` action would be accessed as `Vonage::Voice::Ncco.notify`. For example, the following would accomplish this in the Ruby SDK:

```ruby
module Vonage
  class Voice::Ncco
    ACTIONS = {
      connect: Vonage::Voice::Actions::Connect,
      conversation: Vonage::Voice::Actions::Conversation,
      input: Vonage::Voice::Actions::Input,
      notify: Vonage::Voice::Actions::Notify,
      record: Vonage::Voice::Actions::Record,
      stream: Vonage::Voice::Actions::Stream,
      talk: Vonage::Voice::Actions::Talk
    }

    ACTIONS.keys.each do |method|      
      self.class.send :define_method, method do |attributes|
        ACTIONS[method].new(**attributes).action
      end
    end

    def self.method_missing(method)
      raise ClientError.new("NCCO action must be one of the valid options. Please refer to https://developer.nexmo.com/voice/voice-api/ncco-reference#ncco-actions for a complete list.")
    end

    def self.create(*actions)
      actions.flatten!
    end
  end
end
```

The NCCO builder MUST NOT be part of the SDK client instantiation. Adding the NCCO builder to the client could cause confusion for users as the expectation for client methods is a direct interaction with the Vonage APIs. In this regard, it SHOULD be similar to the JWT generator as specified in its [RFC](https://raw.githubusercontent.com/Nexmo/server-sdk-specification/main/rfcs/accepted/RFC-1-server-sdk-jwt-packages.md).

The NCCO builder MUST return a fully constructed NCCO in the form of an array, with each action represented as a hash inside the array.

### Field Specifications

The NCCO builder's method and parameter names SHOULD conform to the NCCO naming as much as possible. When a name is not possible or is not the convention in the language of the SDK, the name can be adapted for the language specifications of that specific SDK.

#### Record

| Parameter | Description                                 | Data Type | Default Value                          | Required |
| --------- | ------------------------------------------- | --------- | -------------------------------------- | -------- |
| `format`  | Record the call in a specific format        | string    | `mp3` or `wav` if more than 2 channels | No       |
| `split`   | Record the send and received audio in separate channels | string | none | No |
| `channels` | Number of channels to record (max `32`), `conversation` must also be enabled | integer | none | No |
| `endOnSilence` | Stop recording after `n` seconds of silence | integer | none | No |
| `endOnKey` | Stp recording when a digit is pressed, possible values are `*`, `#` or a single digit between `0-9` | string | none | No |
| `timeOut` | Max length of a recording in seconds, between `3` and `7200` | integer | none | No |
| `beepStart` | Set to `true` to play a beep when recording starts | boolean | `false` | No |
| `eventUrl` | URL for the webhook endpoint | string | none | No |
| `eventMethod` | HTTP method for request to `eventUrl` | string | `POST` | No |

#### Conversation

| Parameter | Description                                 | Data Type | Default Value                          | Required |
| --------- | ------------------------------------------- | --------- | -------------------------------------- | -------- |
| `name` | Name of the conversation | string | none | Yes |
| `musicOnHoldUrl` | URL to `mp3` file to stream to participants before conversation starts, `startOnEnter` must be set to `false` for this to be enabled | string | none | No |
| `startOnEnter` | Conversation starts when first participant joints | boolean | `true` | No |
| `endOnExit` | Conversation ends when moderator hangs up | boolean | `false` | No |
| `record` | Record the conversation | boolean | `false` | No |
| `canSpeak` | An array of leg UUIDs that participant can be heard by, if not provided can be heard by all. An empty list will silence them for everyone. | array | none | No |
| `canHear` | An array of leg UUIDs that participant can hear, if not provided can hear all. An empty list will silence everyone for them. | array | none | No |
| `mute` | Mute the particpant, default `false` | boolean | `false` | No |


#### Connect

| Parameter | Description                                 | Data Type | Default Value                          | Required |
| --------- | ------------------------------------------- | --------- | -------------------------------------- | -------- |
| `endpoint` | An array of one endpoint to connect to, see [possible endpoints](https://developer.vonage.com/voice/voice-api/ncco-reference#endpoint-types-and-values) | array | none | Yes |
| `from` | A number in `E.164` format that identifies the caller | string | none | No |
| `eventType` | Set to `synchronous` to make the action synchronous | string | none | No |
| `timeout` | Number of seconds call rings for if unanswered. Default is `60` | integer | `60` | No |
| `limit` | Max length of call in seconds | integer | `7200` | No |
| `machineDetection` | Behavior when API detects an answering machine, either `continue` or `hangup` | string | none | No |
| `eventUrl` | URL for webhook event data | array | none | No |
| `eventMethod` | HTTP method for the event URL | string | `POST` | No |
| `ringbackTone` | A URL that points to a tone to played on repeat to caller while ringing | string | none | No |

#### Talk

| Parameter | Description                                 | Data Type | Default Value                          | Required |
| --------- | ------------------------------------------- | --------- | -------------------------------------- | -------- |
| `text` | A string up to 1,500 characters containing the TTS message | string | none | Yes |
| `bargeIn` | Action is terminated when user interacts either with DTMF or ASR input if set to `true` | boolean | `false` | No |
| `loop` | Number of times the audio is repeated before call is closed | integer | `1` | No |
| `level` | Audio level for stream between `-1` and `1` with a precision of `0.1` | integer | `0` | No |
| `language` | The language for the message you are sending | string | `en-US` | No |
| `style` | The vocal style | integer | `0` | No |


#### Stream

| Parameter | Description                                 | Data Type | Default Value                          | Required |
| --------- | ------------------------------------------- | --------- | -------------------------------------- | -------- |
| `streamUrl` | URL to an mp3 or wav audio file | array | none | Yes |
| `level` | Audio level for stream between `-1` and `1` with a precision of `0.1` | integer | `0` | No |
| `bargeIn` | Action is terminated when user interacts either with DTMF or ASR input if set to `true` | boolean | `false` | No |
| `loop` | Number of times the audio is repeated before call is closed | integer | `1` | No |


#### Input

| Parameter | Description                                 | Data Type | Default Value                          | Required |
| --------- | ------------------------------------------- | --------- | -------------------------------------- | -------- |
| `type`    | Input type: `['dtmf']`, `['speech']` or `['dtmf', 'speech']` | array | none | Yes |
| `dtmf` | See [NCCO Reference](https://developer.vonage.com/voice/voice-api/ncco-reference#dtmf-input-settings) | hash | none | No |
| `speech` | See [NCCO Reference](https://developer.vonage.com/voice/voice-api/ncco-reference#dtmf-speech-settings) | hash | none | No |
| `eventUrl` | Send webhook events to this URL | string | none | No |
| `eventMethod` | HTTP method to use for `eventUrl` | string | `POST` | No |

#### Notify

| Parameter | Description                                 | Data Type | Default Value                          | Required |
| --------- | ------------------------------------------- | --------- | -------------------------------------- | -------- |
| `payload` | The JSON body to send to your event URL | JSON | none | Yes |
| `eventUrl` | The URL to send to events to | string | none | Yes |
| `eventMethod` | HTTP method to use when sending `payload` | string | `POST` | No |

## Record of Votes

## Resolution

