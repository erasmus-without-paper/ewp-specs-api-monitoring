Monitoring API
==============

* [What is the status of this document?][statuses]
* [See the index of all other EWP Specifications][develhub]


Summary
-------

This document describes the **Monitoring API** implemented by the EWP Stats Portal, which is hosted by EWP administrators.
This API allows clients in the EWP network to inform the network's administrators about any issues encountered when making
requests. It SHOULD NOT be implemented by anyone other than the EWP Stats Portal. You will be only using it as a client.


Introduction
------------

The distributed design of the EWP network, while flexible, makes it hard to control and verify interactions happening
in the network. Errors could arise for a variety of reasons: bugs in the implementation, incorrect data, or unreachable
servers. In order to minimize the number of errors in the long term, and to fix new ones quickly, members of the EWP network
should actively report any incorrect behaviours. We decided that it's the clients who should take up this responsibility,
as they are the ones initializing interactions, and they are able to validate responses and detect network errors.


When to call
------------

Clients SHOULD call this API upon detecting any problem with communication with any of the servers in the EWP network,
except for:
* Discovery or Echo API requests,
* calls to the EWP Stats Portal or the Registry Service.

A general rule of thumb is to call this API every time you perform error handling or catch an exception when making a request.
This includes:
* Not being able to find an API in the catalogue, provided that the server is actually expected to implement it
  (for example: the server sent a notification about a new LA, but doesn't implement the API for the client to download it).
* No response received from the server, like in the case of a DNS failure, SSL certificate error, request timeout, etc.
* HTTP error code returned by the server.
* Failed authentication or validation of the server's response.


Server implementing this API
----------------------------

The server implementing this API is called EWP Stats Portal. Its identifier in the EWP network is:

```
stats.erasmuswithoutpaper.eu
```

If the server is not available, clients are not required to call this API repeatedly or store messages until further time.
A simple "call and forget" is good enough.


Request method
--------------

Requests MUST be made with HTTP POST method.


Request parameters
------------------

Parameters MUST be provided in the regular `application/x-www-form-urlencoded` format.


### `server_hei_id` (required)

Identifier of the HEI the client tried to communicate with.


### `api_name` (required)

Name of the API the client tried to use. It MUST be the same as the name of the corresponding API element in the catalogue.
For example, for the [Outgoing Mobility Learning Agreements API][omobility-las-api], it would be `omobility-las`.
It MUST NOT be any of the following: `discovery`, `echo`, `monitoring`, `registry`.


### `endpoint_name` (optional)

Name of the endpoint the client tried to call. It MUST be the same as the name of the corresponding URL element in the catalogue,
without the `-url` suffix. For example, possible values for the [Interinstitutional Agreements API][iias-api]
are `get`, `index`, and `stats`. If the endpoint does not have a name, i.e. its URL element is called `url`,
then this parameter MUST be empty.


### `http_code` (optional)

HTTP status code received from the server. This parameter MUST be passed if the server returned a response.
Otherwise, it MUST be empty.


### `server_message` (optional)

Error description received from the server. This parameter MUST be passed if the server returned a *4xx* or *5xx* status code
along with an `<error-response>` element, in which case the passed value MUST be the content of the `<developer-message>` element,
as defined in the [common-types.xsd][common-types] file. The parameter MUST be empty otherwise.


### `client_message` (optional)

Error description created by the client. This parameter MAY be empty if the server returned a *4xx* or *5xx* status code.
Otherwise, it MUST be passed, and it SHOULD describe the problem as clearly as possible. It MAY be enough to use the message
from an exception. The message MUST NOT contain any private data.


Response
--------

The server will respond with an XML document whose format is described in the [response.xsd](response.xsd) file.


What happens after a call
-------------------------

Errors reported using the Monitoring API can be viewed on the [Monitoring page of the EWP Stats Portal][stats-portal-monitoring].

Additionally, the system sends periodic reports, containing a summary of issues encountered by the clients,
to the server administrators, using their email addresses from the `<admin-email>` elements in the catalogue.


Privacy
-------

Reported data will be publicly available. This is why the error reports SHOULD NOT contain any private data.

However, it must be acknowledged that clients do not have full control over some of the data required by this API,
such as the `server_message` parameter. To minimize the risk of personal data leakage, the EWP Stats Portal
will make every attempt to obfuscate identifiers that are common in the EWP Network, such as email addresses,
phone numbers, and European Student Identifiers.


Security
--------

This version of this API uses [standard EWP Authentication and Security, Version 2][sec-v2].
Server implementers choose which security methods they support by declaring them in their Manifest API entry.


[common-types]: https://github.com/erasmus-without-paper/ewp-specs-architecture/blob/stable-v1/common-types.xsd
[develhub]: http://developers.erasmuswithoutpaper.eu/
[iias-api]: https://github.com/erasmus-without-paper/ewp-specs-api-iias/blob/stable-v6/
[omobility-las-api]: https://github.com/erasmus-without-paper/ewp-specs-api-omobility-las/tree/stable-v1
[sec-v2]: https://github.com/erasmus-without-paper/ewp-specs-sec-intro/tree/stable-v2
[stats-portal-monitoring]: https://stats.erasmuswithoutpaper.eu/monitoring/
[statuses]: https://github.com/erasmus-without-paper/ewp-specs-management#statuses
