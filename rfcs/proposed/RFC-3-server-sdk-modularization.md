# Server SDK Modularization

## Problem

Unlike many SaaS (Software-as-a-Service) companies, Vonage operates as a CPaaS (Communications-Platform-as-a-Service). The key differentiator is that Vonage ships various products instead of focusing on a single product. While a competitor like [Agora](https://www.agora.io/en/) may ship a single product, our Video API is but one of many products our SDKs support. Each of these products moves under its own timeline as defined by its product managers and business units. This can lead to heavy development in one product while other products are untouched.

As the SDKs support various products, we should find a way to allow us to ship individual beta components while making maintenance easy.

## Duration
Proposal - 2022-06-16 - 2022-06-30

## Current State
Proposed

## Proposers
Chris Tankersley

## Detail

The Server SDKs should expose as much of our API base as possible. With the current delineations between general availability, beta, and developer preview APIs it can be very hard to publish non-GA code to our customers. While we do offer Developer Preview code in our NodeJS SDK, and Beta code in our PHP SDK (in some cases), the actual usage and need can be inconsistent. For example, at one point various parts of Conversation were added to a handful of the SDKs as GA code without the API being officially GA.

One solution to this problem is to break the SDKs up into multiple modules. Individual modules can then have their individual full releases, betas, and alphas that correspond to the effort the business wants to put into a single language. This also allows us to work help work with specific languages that may see a spike in usage for a product outside of our normal Definition of Done. As an example, the Definition of Done says that Beta product support includes PHP and NodeJS for Vonage APIs, but for Video one of the heaviest used SDKs was Python. We should be flexible enough to provide beta and alpha packages as needed by our customers.

Another pain point with the current system is it makes it much harder to ship "beta" versions of products to customers. For example, our NodeJS SDK has a massive `beta` branch that attempts to keep and reconcile every product beta feature. This has led to a branch that is both ahead and behind the mainline releases. Moving a feature from `beta` to `main` is a manual process to extract, port, and resync the two branches.

## Proposed Solution
Many languages use dependency resolution systems to handle multiple dependencies in a project. We can utilize those frameworks to ship product-specific modules that can then ship independently. 

Each SDK MUST supply the following modules:
* Core, comprising of:
    * Application Management
    * Number management
* Messages
* Number Insight
* SIP
* SMS
* Verify
* Voice

Each SDK MAY supply the following modules as beta modules:
* Conversation
* Core, comprising of:
    * Redact
    * Reports
    * Audit API

Each module MUST be self-contained. For example, a user MUST be able to create a client that interacts with the SMS API, and a separate client to interact with the Voice API. Additional modules can be added to the `core` project than are shareable across modules, like authentication.

### Module Names
Modules MUST supply packages using either of the following naming conventions, depending on language and package manager conventions:

* `@vonage/<snake-case-product-name>`
* `vonage/<snake-case-product-name>`
* `vonage-<snake-case-product-name>`

For example, PHP would supply the following Generally Available packages:

* `vonage/core`
* `vonage/messages`
* `vonage/number-insight`
* `vonage/sip`
* `vonage/sms`
* `vonage/verify`
* `vonage/voice`

NodeJS would supply the following packages:
* `@vonage/core`
* `@vonage/messages`
* `@vonage/number-insight`
* `@vonage/sip`
* `@vonage/sms`
* `@vonage/verify`
* `@vonage/voice`

### Module Versioning
Each module will maintain its own version numbers, but every module MUST follow [SemVer](https://semver.org/). While the SDKs currently support semver, additional levels of versioning must be maintained for "Developer Preview" and "Beta" releases of modules. 

* GA releases MUST be tagged in `MAJOR.MINOR.PATCH`
* GA Release Candidates MUST be tagged in `MAJOR.MINOR.PATCH-rc.X`
* Beta releases MUST be tagged in `MAJOR.MINOR.PATCH-beta.X`
* Developer Preview releases MUST be tagged in `MAJOR.MINOR.PATH-alpha.X`

For SDKs with existing code that is being turned into a module, the new module SHOULD carry over the existing version number. For example, if the PHP client is at `vonage/client-core:3.1.0`, each new module MAY begin at `3.1.0`. Brand new modules MUST start at `1.0.0-(rc|beta|alpha).1`, as specified by the product's current release label.

### Module Github Repositories

The overall structure of the modules can be left up to the language. For example, NodeJS allows multiple packages to be housed under a single repository as a monorepo, where PHP packages tend to be one package per repository. 

If the language supports multiple packages under a single repository, the SDK MUST keep the current naming convention: `https://github.com/vonage/vonage-<language>-sdk`.

If the language prefers to break modules into separate repositories, the SDK MUST append the module name to the normal naming convention: `https://github.com/vonage/vonage-<language>-sdk-<product-name>`.

For example:

* NodeJS: `https://github.com/vonage/vonage-node-sdk`
* PHP:
  * `https://github.com/vonage/vonage-php-sdk-core`
  * `https://github.com/vonage/vonage-php-sdk-video`
  * `https://github.com/vonage/vonage-php-sdk-messages`

### Full Release Package

SDKs MUST supply a package that includes all Generally Available products as a single installable package. This allows new users to quickly get up-to-speed by installing a single package and provides an experience that matches our current monolithic SDKs.

These packages MUST follow the existing names for the packages.

* NodeJS: `@vonage/server-sdk`
* PHP: `vonage/client`
* Ruby: `vonage`
* Python: `vonage`
* Java: `com.vonage.client`
* .NET: `Vonage`

These releases MUST provide a common interface to access the sub-modules, as they do under the monolithic packages. 

```php
$client = new Vonage\Client(
    new Vonage\Client\Credentials\Basic(API_KEY, API_SECRET)
);
$client->sms->send(...);
$client->voice->createOutboundCall(...)
```

```javascript
const vonage = new Vonage({
    apiKey: API_KEY,
    apiSecret: API_SECRET,
);
vonage.message.sendSMS(...);
vonage.calls.create(...);
```

## Record of Votes

## Resolution

