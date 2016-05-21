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

- Must support API credentials (key and secret) for authentication.
- Must support OAuth1 token for authentication.
- Must require an authentication method (credentials or oauth) for construction.
- Must allow override of the base URL(s) (currently there are multiple base urls).
- Must allow a shared secret (for WebHook request signatures).
- Must not force a singleton (but may provide access to a singleton).
 
###HTTP Client
A client library will be making HTTP requests to Nexmo's API. Client libraries:

- Must use a well supported HTTP client.
- Must use a HTTP client that supports proxies.
- Should use the official/runtime provided HTTP client if available.
- Should allow a user defined HTTP client.
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
- Should have a type that classifies the error
 	* Transport error (could not reach the API).
 	* API Errors (error response from API).
 	* Validation Errors (unexpected data from API).

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
- Should report language version in each request. 
- Should set a user-agent with the following format: LIBRARY-NAME/LIBRARY-VERSION/LANGUAGE-VERSION