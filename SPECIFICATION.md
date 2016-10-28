Nexmo Client Library Functional Specification
=============================================
 
Overview
--------
The goal of this document is to make all Nexmo Client Libraries have a similar feel, while still adopting the standard 
practice and usage of each individual language. This is a functional specification based on the user's interaction with 
the client library.

- This document is a work in progress, and should be changed and updated as changes are made to the APIs or developer 
  needs are discovered.
- Currently there is no versioning this document or guidelines to version libraries based on their adoption of this 
  specification. There should be.
- Client Libraries attempt to make the API and the data it provides simple and easy to use. They do not attempt to 
  solve specific use cases.
- This specification only applies to languages that are object oriented.
 
Reasonings
----------
Some of the concepts behind the actual specification. 

- Leveraging existing HTTP Clients makes it possible for users to call the API directly when needed, with auth handled 
  by the client library.
- Using entities / objects instead of simple hashes / arrays allows IDEs to autocomplete, creates less dependence on 
  API reference, can normalize some inconsistencies.
- Using entities / objects allows easy reuse of parameter sets. 
- Using entities / objects potentially helps with storage of related data.
- Providing parsed objects (not just raw response) removes the need for users to implement complex error checking / 
  parsing.
- Leveraging errors and exceptions as provided by a language removes the need for users to implement complex error 
  checking / parsing.
- Providing support for callbacks enables easy signature support and removes verbose sanity checks on inbound data 
  before it can be used.
- Providing example requests / responses (of both successful and failed requests) allows testing to be isolated to the 
  client, not an integration test that involves the API.
- Sharing entities / objects between API methods and callbacks allows reuse in cases like a reply or proxy, or building 
  of more complex entities in cases like messages and DLRs, or verifications.
 
Specification
-------------
### Creating a Client
A developer will create a client object with some configuration values. This configuration:

- MUST support multiple authentication credentials (see '[Authenticating Requests](authenticating-requests)')
- MUST require at least one authentication credential for construction.
- MUST allow override of the base URL(s) (currently there are multiple base urls).
- MUST not force a singleton (but may provide access to a singleton).
 
### Authenticating Requests
The current state of the API allows multiple authentication credentials. The client library:

- MUST allow a combination (one or more) of the following credential types:
    * MUST allow an API Key and Secret (used for _most_ requests, being replaced by JWT)
    * SHOULD allow a signature secret (used for _some_ API requests, and to validate WebHook signatures)
    * MUST allow a private key for JWT authentication (used for _some requests, eventually replacing key and secret)
    * MAY allow OAuth1 set of tokens for authentication
- MUST allow all credential types as string values, may also support alternative values (path to file, etc).
- MUST NOT allow multiple credentials of the same type (two private keys, two sets of key and secret, etc).
- MUST determine what type of authentication credential should be used for a request.
- SHOULD prefer authentication types in this order, when an API supports more than one:
    * JWT
    * OAuth
    * Signature Secret
    * Key / Secret
- MAY allow choosing a type of authentication credential when an API supports more than one. 
- MAY internally generate a short lived JWT per request.
- MAY internally generate a long lived JWT for multiple requests.
- MUST provide a method to generate a JWT:
    * MUST allow user provided application id
    * MUST allow user defined parameters in the token body.
    * MUST provide default timestamps the JWT validity (`nbf`) and expiration (`exp`) and issued (`iat`).
    * MUST provide default value for the unique JWT identifier (`jti`).
    * MUST only use defaults if not defined by the user.
 
###HTTP Client
A client library will be making HTTP requests to Nexmo's API. Client libraries:

- SHOULD use the official/runtime provided HTTP client if available.
- SHOULD use a well supported HTTP client.
- SHOULD allow a user defined / configured HTTP client.
- SHOULD allow user created HTTP requests to be sent.
    * This allows use of new APIs / legacy APIs without explicit support.
    * This adds common request properties like authorization, user agent, etc.
- MAY allow HTTP requests to be created but not sent.
- MUST be mockable / testable.
    * This allows testing of HTTP and API error handling.
    * This allows testing of rich objects as responses.
 
###API Methods
When a developer makes an API call through the client library, the interface for that call:

- MUST be represented as a namespaced method of the client library.
- SHOULD namespace following the path structure of the API:
    * `POST /calls/`: `client.calls.post(data)`
    * `GET /calls/1234`: `client.calls('1234').get`
    * `PUT /calls/1234`: `client.calls('1234').put(data)`
    * `PUT /calls/1234/stream`: `client.calls('1234').stream.put(data)`
    * `DELETE /calls/1234/talk`: `client.calls('1234').talk.delete()`
    * `PUT /conversations/1234/members/5678`: `client.conversations('1234').members('5678').put(data)`
- MAY namespace with resource ID as parameter (if recommended fluent-like namespacing is not possible / practice):
    * `GET /calls/1234`: `client.calls.get('1234')`
    * `PUT /calls/1234`: `client.calls.put(data, '1234')`
    * `PUT /conversations/1234/members/5678`: `client.conversations.members.put(data, '1234', '5678')`
    * `PUT /conversations/1234/members/5678`: `client.conversations.members.put(data, ['1234', '5678'])`
- MAY also be represented as methods of an entity object returned by the client:
    * `PUT /calls/1234/stream`: `client.calls.get('1234').stream.put(data)`     
- MAY alias HTTP method names to CRUD for configuration resources:
    * `POST /applications/`: `client.applications.create(data)`
    * `PUT /applications/1234`:  `client.applications.update(data)`
- MAY alias HTTP method names to natural language for transactional resources:
    * `POST /verifications/`: `client.verifications.start(data)`
- SHOULD provide convenience methods for transactional resources:    
    * `PUT /calls/1234`: `client.calls('1234).hangup()`
    * `PUT /calls/1234`: `client.calls('1234).transfer(url)`
- MAY alias `get()` to the namespaced method:
    * `client.calls.get('1234)` = `client.calls('1234')`
    * `client.calls.get('1234)` = `client.calls('1234')`
- SHOULD allow optional rate limiting.
    * SHOULD gracefully handle rate limit errors by default.
    * MAY optimistically avoid rate limit with a configurable self imposed rate limit.
    * MUST allow rate limiting to be turned off.
- MUST accept arguments as input.
- SHOULD accept entity objects as input:
    * MUST use interface based entity types as inputs.
 
###Entities
API calls can create, manipulate, and retrieve entities (resources, objects, things, etc). Client libraries should 
represent these entities in a concrete way. API entities:

- MUST be represented as objects.
- MAY be reduced to an array or hash.
- MAY provide the interface of an array or hash.
- MUST be shared between API requests, responses, and webhooks.
- SHOULD support the creation of related entities.
    * A message entity could create another message entity as a reply.
    * A canceled verification entity could create a new attempt.
- MUST be extendable and mutable.
    * If provided an entity as input, MUST use that entity as output.
    * MAY provide configuration to define the entity class the client creates.
- MUST only be required as interfaces.

###API WebHooks
When Nexmo sends a WebHook to the developer's application, the client library:

- MUST support checking the request signature.
- SHOULD provide a way to create entity objects from the request.
    * SHOULD default to using standard parameters / native objects.
    * MUST allow those parameters to be provided for testing.
- SHOULD validate and normalize the expected properties.
    * SHOULD raise an exception or error if properties are invalid.
    * MUST allow the exception or error to be suppressed.
    * MUST normalize the values into proper types.
 
###Errors
Interactions with the API may result in an error, these errors:
 
- MUST be wrapped in native errors or exceptions.
- SHOULD have a type that classifies the general error:
    * `Transport` Errors (could not reach the API, did not receive a response from the API).
    * `Client` Errors (invalid data, bad credentials, API rejected due to invalid request, 400 class errors).
    * `Server` Errors (internal API error, could not complete the request, 500 class errors).
    * `Unexpected` Errors (unexpected data from API).
- SHOULD unify presentation of errors, regardless of API:
    * Legacy APIs return a status of `200` with an error body.
    * New APIs should return the proper status, with error details in the body.
- SHOULD pass on specific error codes and error message from API.
- SHOULD provide access to the request / response objects.

###Logging
Troubleshooting failed API requests can be difficult, so access to logs are invaluable. Logging:

- SHOULD use language / runtime standard log if available and generally used.
- MUST be configurable:
  - MUST allow enabling / disabling.
  - MAY allow verbosity to be defined.
- MAY delegate configuration to a logging interface if generally used.

###Testing
As developers rely on a working client library, it should be both tested and testable. Client libraries:

- SHOULD be fully unit tested.
- SHOULD expect test as a part of any pull request.
- SHOULD be mockable.
- SHOULD use API specification as source of mock requests and responses.
    * When API specification is not available, this specification may be used as the repository for test data.
    
###Reporting
To better understand the usage of needs of developers building on Nexmo, libraries:

- MUST identify requests as originating from the library.
- MUST report internal client library version in each request.
- SHOULD report language version in each request, if not possible MUST report version as `-`
- MUST set a user-agent with the following format: `LIBRARY-NAME/LIBRARY-VERSION/LANGUAGE-VERSION`
- MUST allow an application name and version to be appended with the following format: `/APP-NAME/APP-VERSION`