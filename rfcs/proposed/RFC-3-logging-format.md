# Logging Format

## Problem

Each Vonage SDK caters to a different programming language and its individual quirks. At the heart of each SDK is the simple fact that it is performing HTTP requests back to a common API. While the way each SDK performs this request is, and should be, different depending on the language, how they record errors should be consistent.

## Duration

General Request for Comments - 2021-05-19 - 2021-06-02

Implementation in two SDKs - 2021-06-03 - 2021-06-17

Final Voting - 2021-07-01 - 2021-07-04

## Current State

General Request for Comments

## Proposers

Chris Tankersley <chris.tankersley@vonage.com>

## Detail

Currently each SDK allows a logging mechanism to be passed in language-appropriate ways, but the output of the logs can vary wildly from SDK to SDK. For example the Python SDK has various logging calls throughout different methods, but the PHP SDK only logs requests and responses. To further complicate matters, each SDK logs out to a different format.

In order to facilitate a more cohensive experiance for developers, each SDK should output to a common format with a common set of information. Parsers can then be generated that can read and extract data from any of the SDK logs. Users can then expect a base amount of information from each log entry.

## Proposed Solution

Each log entry:
* MUST be on a single line
* MUST be in JSON format
* MUST contain a severity
* MUST contain a datetimestamp
* MUST contain the API Product that generated the error
* MUST contain the SDK version
* MUST contain the Language
* MUST contain the Language version
* MUST contain only string values, with the `additional-information` key being an exception
* SHOULD contain a base route being accessed
* SHOULD contain the original error code returned from the API
* SHOULD contain the original error message returned from the API
* MAY contain additional debugging info
* Additional debugging information MUST be a JSON key-value object of <string, string>

```json
// Condensed line
{"severity":"debug","timestamp":"2021-05-18 17:46:00","api":"sms","sdk-version":"2.8.0","language":"php","language-version":"7.4.5","route":"https://rest.nexmo.com/sms/json","error-code":"1","error-message":"Throttled. You are sending SMS faster than the account limit.","additional-information":{"sms-type":"sms","from":"15556660000"}}

// Expanded for readability
{
  "severity": "debug",
  "timestamp": "2021-05-18 17:46:00",
  "api": "sms",
  "sdk-version": "2.8.0",
  "language": "php",
  "language-version": "7.4.5",
  "route": "https://rest.nexmo.com/sms/json",
  "error-code": "1",
  "error-message": "Throttled. You are sending SMS faster than the account limit.",
  "additional-information": {
    "sms-type": "sms",
    "from": "15556660000"
  }
}
```

### Fields

| Field | Key | Description | Example |Values |
|-------|-----|-------------|---------|-------|
| Severity | `severity` | Severity of the event | `debug` | `emerg`, `alert`, `crit`, `err`, `warning`, `notice`, `info`, `debug` |
| Timestamp | `timestamp` | An ISO 8601-1:2019 timestamp containing the date, hours, minutes, and seconds | `2021-05-18 17:46:00` | |
| API Product | `api` | API that was being accessed | `sms` | `account`, `audit`, `application`, `conversation`, `conversion`, `dispatch`, `number-insight`, `pricing`, `redact`, `messaging`, `numbers`, `reports`, `verify`, `voice`, `sms`, `video`, `pricing`, `media` |
| SDK Version | `sdk-version` | Version of the SDK being used | `2.6.0` | |
| Language | `language` | Language being used | `php` | |
| Language Version | `language-version` | Version of the language being used | `7.4.3` | |
| API Route | `route` | General route being accessed, without IDs | `https://rest.nexmo.com/sms/json` | |
| Error Code | `error-code` | Error code or `type` returned in an error | `1`  | |
| Error Message | `error-message` | Error message or `title` returned in an error | `Throttled` | |
| Additional Debug Information | `additional-information` | Key-Value JSON object containing any additional useful information | `{}` | |

## Record of Votes

## Resolution

