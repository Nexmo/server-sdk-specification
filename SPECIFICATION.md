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

- Must support multiple authentication credentials (see '[]Authenticating Requests](authenticating-requests)')
- Must require at least one authentication credential for construction.
- Must allow override of the base URL(s) (currently there are multiple base urls).
- Must not force a singleton (but may provide access to a singleton).
 
### Authenticating Requests
The current state of the API allows multiple authentication credentials. The client library:

- Must allow a combination (one or more) of the following credential types:
    - Must allow an API Key and Secret (current API)
    - Must allow a signature secret (use for _some_ API requests, and to validate WebHook signatures)
    - Must allow multiple private keys for JWT authentication (replacing key and secret)
    - May allow OAuth1 token for authentication
- Must not allow more than one credential per type.
- Must determine what type of authentication credential should be used for a request.
- May allow choosing a type of authentication credential when an API supports more than one.
- May generate a short lived JWT per request.
- May generate a longer loved JWT for multiple requests.
- Must provide a method to generate a JWT:
    - Must allow user provided application id.
    - Must allow optional user provided valid timestamps (`nbf` and `exp`)
 
###HTTP Client
A client library will be making HTTP requests to Nexmo's API. Client libraries:

- Should use the official/runtime provided HTTP client if available.
- Should use a well supported HTTP client.
- Should when possible allow a user defined HTTP client.
- Should allow HTTP requests to be created but not sent.
    * This is not the standard usage.
    * This allows a developer to interact with the request if needed.
    * This sets up as much of the request as possible (authorization, base urls, etc).
- Must be mockable / testable.
    * This allows testing of HTTP and API error handling.
    * This allows testing of rich objects as responses.
 
###API Methods
When a developer makes an API call through the client library, the interface for that call:

- Must be represented as a method of the client library.
- May also be represented as methods of an entity object returned by the client.
- Must be namespaced by concept (SMS, voice, verification, etc).
- Should allow optional rate limiting.
    * Gracefully handle rate limit errors (enabled by default).
    * Optimistically avoid rate limit (self imposed rate limit).
- Must accept entity objects as input.
- Must accept arguments as input.
- Must use interface based types as inputs.
 
###API WebHooks
When Nexmo sends a WebHook to the developer's application, the client library:

- Must provide a way to create entity objects from the request.
    * Should default to using standard parameters / native objects.
    * Must allow those parameters to be provided for testing.
- Must support checking the signature.
- Must validate and normalize the expected properties.
    * Should raise an exception or error if properties are invalid.
    * Must allow the exception or error to be suppressed.
    * Must normalize the values into proper types.
 
###Entities
API calls can create, manipulate, and retrieve entities (resources, objects, things, etc). Client libraries should 
represent these entities in a concrete way. API entities:

- Must be represented as objects.
- May be reduced to an array or hash.
- May provide the interface of an array or hash.
- Must be shared between API requests, responses, and webhooks.
- Should allow the creation of related entities.
    * A message entity could create another message entity as a reply.
    * A canceled verification entity could create a new attempt.
- Must be extendable.
    * Developers may extend entities to persist them.
    * Developer may extend entities to add additional business logic.
- Must only be required as interfaces.
 
###Errors
Interactions with the API may result in an error, these may be transport errors, server errors, or client side errors. 
Errors:
 
- Must be wrapped in native errors or exceptions.
- Should have a type that classifies the general error:
    * Transport Errors (could not reach the API, did not receive a response from the API).
    * Client Errors (invalid data, bad credentials, API rejected due to invalid request, 400 class errors).
    * Server Errors (internal API error, could not complete the request, 500 class errors).
    * Unexpected Errors (unexpected data from API).
- Should unify presentation of errors, regardless of API:
    * Legacy APIs return a status of `200` with an error body.
    * New APIs should return the proper status, with error details in the body.
- Should pass on specific error codes and error message from API.

###Logging
Troubleshooting failed API requests can be difficult, so access to logs are invaluable. Logging:

- Should use language / runtime standard log if available and generally used.
- Must be configurable:
  - Can be disabled / enabled
  - Verbosity can be defined.
- Can delegate configuration to a logging interface if generally used.


###Testing
As developers rely on a working client library, it should be both tested and testable. Client libraries:

- Should be fully unit tested.
- Should expect test as a part of any pull request.
- Should be mockable.
- Should use API specification as source of mock requests and responses.
    * When API specification is not available, this specification may be used as the repository for test data.
    
###Reporting
To better understand the usage of needs of developers building on Nexmo, libraries:

- Must identify requests as originating from the library.
- Must report internal client library version in each request.
- Should report language version in each request, if not possible must replort version as `-`
- Must set a user-agent with the following format: `LIBRARY-NAME/LIBRARY-VERSION/LANGUAGE-VERSION`
- Must allow an application name and version to be appended with the following format: `/APP-NAME/APP-VERSION`