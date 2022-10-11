Monitoring API
==============

* [What is the status of this document?][statuses]
* [See the index of all other EWP Specifications][develhub]


Summary
-------

This document describes the **Monitoring API** implemented by the EWP Stats Portal, which is hosted by EWP administrators.
This API allows clients in EWP network to inform network's administrators about any issues encountered when making
requests. This API SHOULD NOT be implemented by anyone other than EWP Stats Portal. Clients MUST call this API
in case of detecting any exceptions or incorrect responses from any of the servers in the EWP network,
except EWP Stats Portal and the Registry Service. In case of issues with calling this API, like server unavailability,
clients are not required to call this API repeatedly or store messages until further time.
It means that simple "call and forget" is good enough. Calls to this API MUST NOT contain any private data.


Introduction
------------

The distributed design of EWP network, while flexible, makes it hard to control and verify interactions happening
in the network. Errors could arise for a variety of reasons - bugs in implementation, incorrect data, or unreachable
servers. In order to minimize the number of errors in the long term and fix new ones quickly, members of EWP network
should actively report any incorrect behaviours. We decided clients should take up this responsibility, as they are the
ones initializing interactions, and they are able to verify responses and detect network errors.


Request method
--------------

 * Requests MUST be made with HTTP POST method.


Request parameters
------------------

Parameters MUST be provided in the regular `application/x-www-form-urlencoded`
format.


### `server_hei_id` (required)

HEI ID of the server.


### `api_name` (required)

The name of the API that was called in the request. The name should be the same as the name of the API in the catalogue.
For example for [Outgoing Mobility Learning Agreements API][outgoing-la-api] it's *omobility-las*.


### `error_type` (required)

The type of error that is being reported. This value MUST be one of:

  * API_LOOKUP_ERROR - When there is any error with looking up the API from the catalogue. For example the HEI does not
    implement an API that it is expected to, like the API for fetching an LA when a CNR notification was received.
  * NETWORK_ERROR - For network errors other than timeout, like DNS failures or SSL certificate errors.
  * TIMEOUT - When client decided the server is taking too long to respond.
  * SERVER_ERROR - When server returned any *4xx* or *5xx* HTTP code.
  * INVALID_RESPONSE - When the client could not verify the server's response, for example because of some incorrect
    values returned by the server.


### `api_url` (optional)

URL of the API that was called in the request. This parameter MUST be passed when *error_type* parameter has value other
than *API_LOOKUP_ERROR*.


### `http_code` (optional)

The HTTP status code that was returned by the server in the request that is being reported. This parameter MUST be passed
when *error_type* parameter has value *SERVER_ERROR* or *INVALID_RESPONSE*.


### `message` (optional)

The message containing error description. Its value depends on error type:
  * In case the *error_type* is *SERVER_ERROR*, this parameter is required only if the server returned an `<error-response>`
    and it MUST be its `<developer-message>`, as defined in the [common-types.xsd][common-types] file.
  * In case of other errors, the message MUST be created by the client. It MAY be enough to use the message from an exception,
    if it will be clear for other people.

Handling of invalid parameters
------------------------------

 * General [error handling rules][error-handling] apply.


Response
--------

Servers MUST respond with a valid XML document described by the
[response.xsd](response.xsd) schema. See the schema annotations for further information.


Server implementing this API
----------------------------

The server implementing this API is called EWP Stats Portal. It is identified by HEI id
```
stats.erasmuswithoutpaper.eu
```


When to call
------------

The general rule to calling this API is: Call it in every case you do error handling or exception catching. This implies that
every HTTP status code that is *4xx* or *5xx* should be reported, as well as connection timeouts. However, that is not all.
For example server could return status code 200, but in reality the data returned is lacking, incorrect, or generally
does not make sense. This situation also warrants a call to the monitoring server.

Due to the number of different APIs it's not possible to clearly define every potential incorrect interaction in the network.
It's better to send too much rather than too little, but remember to use your common sense.


What happens after a call
-------------------------

EWP Stats Portal gathers and analyzes all the data sent to it through monitoring API. In some cases it may automatically
decide to email a provider implementing a server that is the source of incorrect interaction, with an information about
a bug and an offer to help. It does not mean that every monitoring API call will end with an email to server provider, just in cases deemed fit.


Privacy
-------

EWP administrators are not authorized to access private data that is being exchanged in the network. That is why calls to
monitoring API MUST NOT contain any private data. Clients should send only the data described in the request parameters.

Security
--------

This version of this API uses [standard EWP Authentication and Security, Version 2][sec-v2].
Server implementers choose which security methods they support by declaring them in their Manifest API entry.


[common-types]: https://github.com/erasmus-without-paper/ewp-specs-architecture/blob/stable-v1/common-types.xsd
[develhub]: http://developers.erasmuswithoutpaper.eu/
[error-handling]: https://github.com/erasmus-without-paper/ewp-specs-architecture#error-handling
[outgoing-la-api]: https://github.com/erasmus-without-paper/ewp-specs-api-omobility-las/tree/stable-v1
[sec-v2]: https://github.com/erasmus-without-paper/ewp-specs-sec-intro/tree/stable-v2
[statuses]: https://github.com/erasmus-without-paper/ewp-specs-management#statuses
