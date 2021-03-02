# Server SDK JWT Packages

## Problem
Many developers, both in and outside of the company, use our APIs without using a full SDK. This can be for many reasons, but authentication is a core issue that all developers using our APIs will face. While key/secret authentication is simple to implement, the ACL-based approach with JWTs provides an additional layer of complexity. There are also competing documentation differences between how JWTs should behave, and a common package can help smooth over those issues.

## Duration
22 May 2020

29 May 2020

## Current State
Approved


## Proposers
Chris Tankersley

## Detail
JWT authentication is being used on newer APIs being deployed by Vonage as it provides a finer grain of control over authorization between clients. New applications being developed using APIs such as Messages or Conversations are increasingly being developed by outside teams such as Pre-Sales, and there is a large need for generating JWTs easily. Vonage currently offeres two solutions:

* `nexmo/cli`, which can generate JWTs
* `nexmo-java-jwt`, which can generate JWTs in Java

These solutions do not cover the full gamut of with internal teams need, and as we can track via API usage we have a large set of customers using our APIs outside of the Java landscape. Many customers do not use our SDKs as they are implementing our Developer Preview or Beta level products, which do not include SDK support. While customers could introduce our SDKs as dependencies, this introduces a large dependency for a small need. Many customers do not want this.

All of our mainline SDKs already support generating JWTs in some capacity, but generally these JWTs are very wide in capability scope or run under the assumption they are used just for the Voice API. Some SDKs expose JWT generation in a fairly clean-room implementation inside the SDK code itself, while others are tightly coupled to their Nexmo Client integrations.

JWTs by themselves are an industry standard (https://jwt.io/) and there are libraries for creating raw JWT objects, but there is additional complexity in crafting the JWT to the expectations of Vonage. By offering a standalone JWT package per-language, we can offer customers a quick development win by making it easy to generate Vonage-compatible JWTs with our requirements, and continue to allow developers and customers the ability to not have to require a “kitchen sink” SDK for one small operation.

## Proposed Solution
### Packaging
Vonage will provide a self contained JWT token generator for each language, that is automatically installed in the SDKs but can also be installed via each language’s package manager. The package will be stored as an open-source project on Github under `nexmo/vonage-jwt-[language]` and will be maintained with the same level of maintenance we provide the full server SDKs.

Where possible, the package should be made available as vonage/jwt in the various package managers, or a name as close as possible to this scheme.

The JWT package will be a dependency for each mainline Server SDK library.

### Technical Specifications
The generator class MUST be instantiable. By providing bespoke generator instantiations for each JWT generator, there is a more obvious settings flow being represented back to the developer. For example, if setExpirationTime() altered the expiration time globally, the developer may end up with a token expiration time period that they were not expecting. If the developer decides to re-use the token generator, the developer can more easily track their way back to see where options had been changed. It is recommended to use the Builder pattern (https://en.wikipedia.org/wiki/Builder_pattern) to help provide a fluent interface for adding additional options to the generator.

The generator class MUST provide a static factory interface for generating tokens. In this case, the base Generator class MUST NOT store any settings information for later invocations, and the Generator class with return only a single string JWT token. For example, if the developer passes an expiration time to the factory for the generation of a JWT token, subsequent calls to the factory method MUST NOT use the previous expiration setting provided. Unless expressly overridden, the factory method MUST use default values for any claim for token generation.

```php
// Standalone object with the Builder Pattern
$generator = new TokenGenerator('d70425f2-1599-4e4c-81c4-cffc66e49a12', file_get_contents(__DIR__ . '/resources/private.key'));
$generator
    ->setTTL(30 * 60)
    ->setPaths([
        '/*/users/**',
        '/*/conversations/**'
    ])
;
$token = $generator->generate();

// Static Factory
$token = TokenGenerator::factory(
    'd70425f2-1599-4e4c-81c4-cffc66e49a12',
    file_get_contents(__DIR__ . '/resources/private.key'), 
    [
        'ttl' => 30 * 60,
        'paths' => [
            '/*/users/**',
            '/*/conversations/**'
    ]
]);
```

The Application ID and Private Key MUST be supplied as constructor parameters when creating any token, as this information is required for signing of the JWT and authentication to the APIs. The Private Key string MUST be passed as the text of the key, and not as a directory path or file pointer to a location. The Private Key MUST be an RSA-256 key.

Developers MUST be able to specify and override the following information (See Field Specifications for more information on each field):

* `jti` - Developer MUST supply a valid UUIDv4 string
* `nbf` - Developer MUST supply an integer representing a Unix Timestamp (UTC+0)
* `ttl` - Developer MUST supply the number of seconds to add to the iat time, which generates the exp value
* `acl->path` - Developer MUST supply a string with the path, and the developer MAY supply ACL options. The Generator MUST allow the developer to set these items in bulk as well as individually.
* `sub` - Developer MUST supply a string name

Fields MUST be able to be accessed via methods on the Token Generator object.

```php
$generator = new TokenGenerator('d70425f2-1599-4e4c-81c4-cffc66e49a12', file_get_contents(__DIR__ . '/resources/private.key'));
$generator->setTTL(30 * 60);
$generator->addPath('/*/users/**');
$generator->addPath('/*/conversations/**', ['methods' => ['GET']]);

// Set paths in one operation
$paths = [
    '/*/users/**',
    '/*/conversations/**' => [
        'methods' => ['GET']
    ]
];
$generator->setPaths($paths); // This will override any existing path info

$expirationTime = $generator->getExpirationTime();
```

Developers MUST NOT be able to modify the following fields:

* `alg` - This MUST be set to RS256
* `typ` - This MUST be set to JWT
* `application_id` - This MUST be supplied as a mandatory field to the generator, and MUST NOT be changed after
* `iat` - This MUST be generated as a Unix Timestamp (UTC+0) at the time the token string is generated
* `exp` - This value MUST NOT be editable by the developer, as it is generated by iat + ttl

If the developer does not specify a JTI, then the Generator MUST create a valid UUIDv4 string on behalf of the developer. The developer MUST be able to query the generator to access what the generated ID is. If the developer supplies a JTI, then the Generator MUST use the supplied JTI.

The Token Generator MUST return a simple string token.

### Dependencies
The Vonage JWT packages SHOULD depend on third party JWT generation libraries for the actual JWT generation. We should not attempt to hand-craft the actual token itself, but provide a nice Vonage-specific wrapper to how we expect JWTs to behave. The goal is to make it easier for developers to create JWT tokens for our APIs, not have a fully home-grown JWT solution.

### Field Specifications
| Claim/Header | Key | Machine Name* | Description | Data Type | Default Value | Required |
| -------------| --- | ------------- | ----------- | --------- | ------------- | -------- |
| Header | alg | Algorithm | Encryption algorithm being used | string | Must be RS256 | Yes | 
| Header | typ | Type | Token structure | string | Must be JWT | Yes | 
| Claim | application_id | ApplicationID | ID of the application being accessed | string | Yes |
| Claim | iat | IssuedAt | Unix Timestamp (UTC+0), in seconds, when the JWT was created | int | Should default to ‘now’ | Yes |
| Claim | jti | JTI | ID of the JWT | string, UUIDv4 | Random UUIDv4 String | Yes |
| Claim | nbf | NotBefore | Unix Timestamp (UTC+0), in seconds, when the JWT becomes valid | int | No |
| - | - | TTL | Integer representing seconds in which determines how long the token should last. Should be used with iat to help generate exp | int | 900 | N/A |
|Claim | exp | - | Unix Timestamp (UTC+0), in seconds, when the JWT expires. Must be at least 30 seconds, maximum of 24 hours, from the iat, and generating using the TTL value. | int | Yes |
| Claim | acl->paths | Path / Paths | Path information for access control to API routes | object | No |
| Claim | sub | Subject | “Subject” or user created and associated with the Nexmo Application | string | No |

* - Machine Names may be converted to camelCase, snake_case, or PascalCase as appropriate for the implementing language.

### Further API JWT documentation:
* General JWT - https://developer.nexmo.com/concepts/guides/authentication#json-web-tokens-jwt
* Conversations JWT - https://developer.nexmo.com/conversation/guides/jwt-acl
* ACLs - JWT Access Control Lists

## Record of Votes
## Resolution
Draft