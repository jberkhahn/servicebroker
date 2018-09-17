# Open Service Broker API (master - might contain changes that are not yet released)

## Table of Contents
  - [API Overview](#api-overview)
  - [Notations and Terminology](#notations-and-terminology)
  - [Changes](#changes)
    - [Change Policy](#change-policy-for-minor-versions)
    - [Changes Since v2.13](#changes-since-v213)
  - [Headers](#headers)
    - [API Version Header](#api-version-header)
    - [Originating Identity](#originating-identity)
  - [Platform to Service Broker Authentication](#platform-to-service-broker-authentication)
  - [URL Properties](#url-properties)
  - [Service Broker Errors](#service-broker-errors)
  - [Content Type](#content-type)
  - [Catalog Management](#catalog-management)
    - [Adding a Service Broker to the Platform](#adding-a-service-broker-to-the-platform)
  - [Synchronous and Asynchronous Operations](#synchronous-and-asynchronous-operations)
    - [Synchronous Operations](#synchronous-operations)
    - [Asynchronous Operations](#asynchronous-operations)
  - [Blocking Operations](#blocking-operations)
  - [Polling Last Operation for Service Instances](#polling-last-operation-for-service-instances)
  - [Polling Last Operation for Service Bindings](#polling-last-operation-for-service-bindings)
  - [Polling Interval and Duration](#polling-interval-and-duration)
  - [Provisioning](#provisioning)
  - [Fetching a Service Instance](#fetching-a-service-instance)
  - [Updating a Service Instance](#updating-a-service-instance)
  - [Binding](#binding)
    - [Types of Binding](#types-of-binding)
  - [Fetching a Service Binding](#fetching-a-service-binding)
  - [Unbinding](#unbinding)
  - [Deprovisioning](#deprovisioning)
  - [Orphans](#orphans)

## API Overview

The Open Service Broker API defines an HTTP(S) interface between Platforms and
Service Brokers.

The Service Broker is the component of the service that implements the Service
Broker API, for which a Platform is a client. Service Brokers are responsible
for advertising a catalog of Service Offerings and Service Plans to the
Platform, and acting on requests from the Platform for provisioning, binding,
unbinding, and deprovisioning.

In general, provisioning reserves a resource on a service; we call this
reserved resource a Service Instance. What a Service Instance represents can
vary by service. Examples include a single database on a multi-tenant server,
a dedicated cluster, or an account on a web application.

What a Service Binding represents MAY also vary by service. In general, creation
of a Service Binding either generates credentials necessary for accessing the
resource or provides the Service Instance with information for a configuration change.

A Platform MAY expose services from one or many Service Brokers, and an
individual Service Broker MAY support one or many Platforms using different URL
prefixes and credentials.

## Notations and Terminology

### Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to
be interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119).

### Terminology

This specification defines the following terms:

- *Application*: Often the entity using a Service Instance will be a piece of
  software, however, this does not need to be the case. For the purposes of
  this specification, the term "Application" will be used to represent all
  entities that might make use of, and be bound to, a Service Instance.

- *Platform*: The software that will manage the cloud environment into which
  Applications are provisioned and Service Brokers are registered. Users will
  not directly provision Services from Service Brokers, rather they will ask
  the Platform to manage Services and interact with the Service Brokers for
  them.

- *Service*: A managed software offering that can be used by an Application.
  Typically, Services will expose some API that can be invoked to perform
  some action. However, there can also be non-interactive Services that can
  perform the desired actions without direct prompting from the Application.

- *Service Broker*: Service Brokers manage the lifecycle of Services. Platforms
  interact with Service Brokers to provision, and manage, Service Instances
  and Service Bindings.

- *Service Offering*: The advertisement of a Service that a Service Broker
  supports.

- *Service Plan*: The representation of the costs and benefits for a given
  variant of the Service, potentially as a tier.

- *Service Instance*: An instantiation of a Service Offering and Service Plan.

- *Service Binding*: Represents the request to use a Service Instance. As part
  of this request there might be a reference to the entity, also known as the
  Application, that will use the Service Instance. Service Bindings will often
  contain the credentials that can then be used to communicate with the Service
  Instance.


## Changes

### Change Policy for Minor Versions

* Existing endpoints and fields MUST NOT be removed or renamed.
* New OPTIONAL endpoints, or new HTTP methods for existing endpoints, MAY be
added to enable support for new features.
* New fields MAY be added to existing request/response messages.
These fields MUST be OPTIONAL and SHOULD be ignored by clients and servers
that do not understand them.

### Changes Since v2.13

* Added GET endpoints for fetching a
  [Service Instance](https://github.com/openservicebrokerapi/servicebroker/blob/v2.14/spec.md#fetching-a-service-instance)
  and
  [Service Binding](https://github.com/openservicebrokerapi/servicebroker/blob/v2.14/spec.md#fetching-a-service-binding)
* Added support for asynchronous Service Bindings
  ([PR](https://github.com/openservicebrokerapi/servicebroker/pull/334))
  and a new
  [last operation endpoint for Service Bindings](https://github.com/openservicebrokerapi/servicebroker/blob/v2.14/spec.md#polling-last-operation-for-service-bindings)
  endpoint
* Added clarity around concurrent updates
  ([PR](https://github.com/openservicebrokerapi/servicebroker/pull/300))
* Added clarity on how Platform's can clean up after a failed provision or bind
  ([PR](https://github.com/openservicebrokerapi/servicebroker/pull/353))
* Added Opaque Bearer Tokens to the
  [Platform to Service Broker Authentication](https://github.com/openservicebrokerapi/servicebroker/blob/v2.14/spec.md#platform-to-service-broker-authentication)
  section
  ([PR](https://github.com/openservicebrokerapi/servicebroker/pull/398))
* Provided guidance for CLI-friendly names
  ([PR](https://github.com/openservicebrokerapi/servicebroker/pull/425))
* Allow for uppercase characters in Service and Service Plan names
  ([PR](https://github.com/openservicebrokerapi/servicebroker/pull/433))
* Clarify that extra fields in requests and responses are allowed
  ([PR](https://github.com/openservicebrokerapi/servicebroker/pull/436))
* Allow an updated `dashboard_url` to be provided when updating a Service
  Instance ([PR](https://github.com/openservicebrokerapi/servicebroker/pull/437))
* Added an [OpenAPI 2.0 implementation](https://github.com/openservicebrokerapi/servicebroker/blob/v2.14/swagger.yaml)
* Allow for periods in name fields
  ([PR](https://github.com/openservicebrokerapi/servicebroker/pull/452))
* Removed the need for Platforms to perform orphan mitigation when receiving an
  `HTTP 408` response code
  ([PR](https://github.com/openservicebrokerapi/servicebroker/pull/456))
* Moved the `dashboard_client` field to
  [Cloud Foundry Catalog Extensions](https://github.com/openservicebrokerapi/servicebroker/blob/v2.14/profile.md#cloud-foundry-catalog-extensions)
* Added a [compatibility matrix](https://github.com/openservicebrokerapi/servicebroker/blob/v2.14/compatibility.md)
  describing which OPTIONAL features in the specification are supported by
  different Platforms
* Added clarity for returning Service Binding information via the GET endpoints
  ([PR](https://github.com/openservicebrokerapi/servicebroker/pull/517))
* Added guidance for supported string lengths
  ([PR](https://github.com/openservicebrokerapi/servicebroker/pull/518))
* Clarified that the `plan_updateable` field affects modifying the Service Plan,
  not parameters ([PR](https://github.com/openservicebrokerapi/servicebroker/pull/519))
* Clarified which Service Plan ID to use for polling the last operation endpoint
  after an update ([PR](https://github.com/openservicebrokerapi/servicebroker/pull/522))
* Clarified Platform behaviour when a dashboard URL is not returned
  ([PR](https://github.com/openservicebrokerapi/servicebroker/pull/527))
* Fixed an incompatible change introduced in v2.12
  ([PR](https://github.com/openservicebrokerapi/servicebroker/pull/540))
* Added clarity around the state of resources after a failure
  ([PR](https://github.com/openservicebrokerapi/servicebroker/pull/541))
* Added [Content Type](https://github.com/openservicebrokerapi/servicebroker/blob/v2.14/spec.md#content-type)
  guidelines

For changes in older versions, see the [release notes](https://github.com/openservicebrokerapi/servicebroker/blob/master/release-notes.md).

## Headers
The following HTTP Headers are defined for the operations detailed in this spec:

| Header | Type | Description |
| --- | --- | --- |
| X-Broker-API-Version* | string | See [API Version Header](#api-version-header). |
| X-Broker-API-Originating-Identity | string | See [Originating Identity](#originating-identity). |

\* Headers with an asterisk are REQUIRED.

### API Version Header

Requests from the Platform to the Service Broker MUST contain a header that
declares the version number of the Open Service Broker API that the Platform
is using:

`X-Broker-API-Version: 2.14`

The version numbers are in the format `MAJOR.MINOR` using semantic versioning.

This header allows Service Brokers to reject requests from Platforms for
versions they do not support. While minor API revisions will always be
additive, it is possible that Service Brokers depend on a feature from a newer
version of the API that is supported by the Platform. In this scenario the
Service Broker MAY reject the request with `412 Precondition Failed` and
provide a message that informs the operator of the API version that is to be
used instead.

### Originating Identity

Often a Service Broker will need to know the identity of the user that
initiated the request from the Platform. For example, this might be needed for
auditing or authorization purposes. In order to facilitate this, the Platform
will need to provide this identification information to the Service Broker on
each request. Platforms MAY support this feature, and if they do, they MUST
adhere to the following:
- For any OSBAPI request that is the result of an action taken by a Platform's
  user, there MUST be an associated `X-Broker-API-Originating-Identity` header on 
  that HTTP request.
- Any OSBAPI request that is not associated with an action from a Platform's
  user, such as the Platform refetching the catalog, MAY exclude the header from
  that HTTP request.
- If present on a request, the `X-Broker-API-Originating-Identity` header 
  MUST contain the identify information for the Platform's user(s) that took 
  the action to cause the request to be sent.

If the Platform chooses to group multiple end-user operations into one request
to the Broker, then the identity information associated with that one request
MUST accurately reflect the desired indentity associated for each individual
change.

The format of the header MUST be:

```
X-Broker-API-Originating-Identity: Platform value
```

`Platform` MUST be a non-empty string indicating the Platform from which
the request is being sent. The specific value SHOULD match the values
defined in the [profile](profile.md) document for the `context.platform`
property. When `context` is sent as part of a message, this value MUST
be the same as the `context.platform` value.

`value` MUST be a Base64 encoded string. The string MUST be a serialized
JSON object. The specific properties will be Platform specific - see
the [profile](profile.md) document for more information.

For example:
```
X-Broker-API-Originating-Identity: cloudfoundry eyANCiAgInVzZXJfaWQiOiAiNjgzZWE3NDgtMzA5Mi00ZmY0LWI2NTYtMzljYWNjNGQ1MzYwIg0KfQ==
```

Where the `value`, when decoded, is:
```
{
  "user_id": "683ea748-3092-4ff4-b656-39cacc4d5360"
}
```

Note that not all messages sent to a Service Broker are initiated by an
end-user of the Platform. For example, during orphan mitigation or during the
querying of the Service Broker's catalog, the Platform might not have an
end-user with which to associate the request, therefore in those cases the
originating identity header would not be included in those messages.

## Vendor Extension Fields

Senders of messages defined by this specification MAY include additional
fields within the JSON objects. When adding new fields, unique prefixes
SHOULD be used for the field names to reduce the chances of conflicts with
future specification defined fields or other extensions.

Receivers of messages defined by this specification that contain unknown
extension fields MUST ignore those fields and MUST NOT halt processing
of those messages due to the presence of those fields. Receivers are under
no obligation to understand or process unknown extension fields.

## Platform to Service Broker Authentication

While the communication between a Platform and Service Broker MAY be unsecure,
it is RECOMMENDED that all communications between a Platform and a Service
Broker are secured via TLS and authenticated. If communications are secured
via TLS, the Platform and Service Broker SHOULD agree whether the Service
Broker will use a root-signed certificate or a self-signed certificate.

Unless there is some out of band communication and agreement between a
Platform and a Service Broker, the Platform MUST authenticate with the
Service Broker using HTTP basic authentication (the `Authorization:` header)
on every request. This specification does not specify how Platform and Service
Brokers agree on other methods of authentication.

Platforms and Service Brokers MAY agree on an authentication mechanism other
than basic authentication, but the specific agreements are not covered by this
specification. Please see the
[Platform Features authentication mechanisms wiki document](https://github.com/openservicebrokerapi/servicebroker/wiki/Platform-Features)
for details on these mechanisms.

If authentication is used, the Service Broker MUST authenticate the request
using the predetermined authentication mechanism, and MUST return a `401 Unauthorized`
response if the authentication fails.

Note: Using an authentication mechanism that is agreed to via out of band
communications could lead to interoperability issues with other Platforms.

## URL Properties

This specification defines the following properties that might appear within
URLs:
- `service_id`
- `plan_id`
- `instance_id`
- `binding_id`
- `operation`

While this specification places no restriction on the set of characters used
within these strings, it is RECOMMENDED that these properties only contain
characters from the "Unreserved Characters" as defined by
[RFC3986](https://tools.ietf.org/html/rfc3986#section-2.3). In other words:
uppercase and lowercase letters, decimal digits, hyphen, period, underscore
and tilde.

If a character outside of the "Unreserved Characters" set is used, then it
SHOULD be percent-encoded prior to being used as part of the HTTP request, per
[RFC3986](https://tools.ietf.org/html/rfc3986#section-2.1).

## Service Broker Errors

When a request to a Service Broker fails, the Service Broker MUST return an
appropriate HTTP response code. Where the specification defines the expected
response code, that response code MUST be used.

For error responses, the following fields are defined:

| Response Field | Type | Description |
| --- | --- | --- |
| error | string | A single word in camel case that uniquely identifies the error condition. If present, MUST be a non-empty string. |
| description | string | A user-facing error message explaining why the request failed. If present, MUST be a non-empty string. |

### Error Codes

There are failure scenarios described throughout this specification for which
the `error` field MUST contain a specific string. Service Broker authors MUST
use these error codes for the specified failure scenarios.

| Error | Reason | Expected Action |
| --- | --- | --- |
| AsyncRequired | This request requires client support for asynchronous service operations. | The query parameter `accepts_incomplete=true` MUST be included the request. |
| ConcurrencyError | The Service Broker does not support concurrent requests that mutate the same resource. | Clients MUST wait until pending requests have completed for the specified resources. |
| RequiresApp | The request body is missing the `app_guid` field. | The `app_guid` MUST be included in the request body. |

Unless otherwise specified, an HTTP status code in the 4xx range MUST result in
the Service Broker's resources being semantically unchanged as a result of
the incoming request message. Additionally, an HTTP status code in the 5xx
range SHOULD result in the Service Broker's resources being semantically
unchanged as a result of the incoming request message. Note, the 5xx error
case is a "SHOULD" instead of a "MUST" because it might not be possible for
a Service Broker to guarantee that it can revert all possible effects of a
failed attempt at the requested operation.

## Content Type

All requests and responses defined in this specification with accompanying
bodies SHOULD contain a `Content-Type` header set to `application/json`.
If the `Content-Type` is not set, Service Brokers and Platforms MAY still
attempt to process the body. If a Service Broker rejects a request due
to a mismatched `Content-Type` or the body is unprocessable it SHOULD
respond with `400 Bad Request`.

## Catalog Management

The first endpoint that a Platform will interact with on the Service Broker is
the service catalog (`/v2/catalog`). This endpoint returns a list of all
services available on the Service Broker. Platforms query this endpoint from
all Service Brokers in order to present an aggregated user-facing catalog.

Periodically, a Platform MAY re-query the service catalog endpoint for a
Service Broker to see if there are any changes to the list of services.
Service Brokers MAY add, remove or modify (metadata, plans, etc.) the list of
services from previous queries.

When determining what, if anything, has changed on a Service Broker, the
Platform MUST use the `id` of the resources (services or plans) as the only
immutable property and MUST use that to locate the same resource as was
returned from a previous query. Likewise, a Service Broker MUST NOT change the
`id` of a resource across queries, otherwise a Platform will treat it as a
different resource.

When a Platform receives different `id` values for the same type of resource,
even if all of the other metadata in those resources are the exact same, it
MUST treat them as separate instances of that resource.

Service Broker authors are expected to be cautious when removing services and
plans from their catalogs, as Platforms might have provisioned Service
Instances of these plans. For example, Platforms might restrict the actions
that users can perform on existing Service Instances if the associated service
or plan is deleted. Consider your deprecation strategy.

Platforms MAY have limits on the length of strings that they can handle or
display to end users, such as the description of a Service or Service Plan. It
is RECOMMENDED that strings do not exceed 255 characters to increase the
likelihood of having compatibility with any Platform.

The following sections describe catalog requests and responses in the Service
Broker API.

### Request

#### Route
`GET /v2/catalog`

#### cURL
```
$ curl http://username:password@service-broker-url/v2/catalog -H "X-Broker-API-Version: 2.14"
```

### Response

| Status Code | Description |
| --- | --- |
| 200 OK | MUST be returned upon successful processing of this request. The expected response body is below. |

#### Body

CLI clients will typically have restrictions on how names, such as service
and plan names, will be provided by users. Therefore, this specification
defines a "CLI-friendly" string as a short string that MUST only use
alphanumeric characters, periods, and hyphens, with no spaces. This will make it
easier for users when they have to type it as an argument on the command line.
For comparison purposes, service and plan names MUST be treated as
case-sensitive strings.

Note: In previous versions of the specification service and plan names
were not allowed to use uppercase characters. However, this requirement was
not enforced and therefore to ensure backwards compatibility with existing
Service Brokers that might use uppercase characters the specification
has been changed.

For backwards compatibility reasons, this specification does not preclude
the use of CLI-unfriendly strings that might cause problems for command line
parsers (or that are not very meaningful to users), such as `-`.
It is therefore RECOMMENDED that implementations avoid such strings.

| Response Field | Type | Description |
| --- | --- | --- |
| services* | array of [Service](#service-object) objects | Schema of service objects defined below. MAY be empty. |

\* Fields with an asterisk are REQUIRED.

##### Service Object

| Response Field | Type | Description |
| --- | --- | --- |
| name* | string | A CLI-friendly name of the service. MUST only contain alphanumeric characters, periods, and hyphens (no spaces). MUST be unique across all service objects returned in this response. MUST be a non-empty string. |
| id* | string | An identifier used to correlate this service in future requests to the Service Broker. This MUST be globally unique such that Platforms (and their users) MUST be able to assume that seeing the same value (no matter what Service Broker uses it) will always refer to this service. MUST be a non-empty string. Using a GUID is RECOMMENDED. |
| description* | string | A short description of the service. MUST be a non-empty string. |
| tags | array of strings | Tags provide a flexible mechanism to expose a classification, attribute, or base technology of a service, enabling equivalent services to be swapped out without changes to dependent logic in applications, buildpacks, or other services. E.g. mysql, relational, redis, key-value, caching, messaging, amqp. |
| requires | array of strings | A list of permissions that the user would have to give the service, if they provision it. The only permissions currently supported are `syslog_drain`, `route_forwarding` and `volume_mount`. |
| bindable* | boolean | Specifies whether Service Instances of the service can be bound to applications. This specifies the default for all plans of this service. Plans can override this field (see [Plan Object](#plan-object)). |
| instances_retrievable | boolean | Specifies whether the [Fetching a Service Instance](#fetching-a-service-instance) endpoint is supported for all plans. |
| bindings_retrievable | boolean | Specifies whether the [Fetching a Service Binding](#fetching-a-service-binding) endpoint is supported for all plans. |
| metadata | object | An opaque object of metadata for a Service Offering. It is expected that Platforms will treat this as a blob. Note that there are [conventions](profile.md#service-metadata) in existing Service Brokers and Platforms for fields that aid in the display of catalog data. |
| dashboard_client | [DashboardClient](profile.md#dashboard-client-object) | A Cloud Foundry extension described in [Catalog Extensions](profile.md#catalog-extensions). Contains the data necessary to activate the Dashboard SSO feature for this service. |
| plan_updateable | boolean | Whether the service supports upgrade/downgrade for some plans. Please note that the misspelling of the attribute `plan_updatable` as `plan_updateable` was done by mistake. We have opted to keep that misspelling instead of fixing it and thus breaking backward compatibility. Defaults to false. |
| plans* | array of [Plan](#plan-object) objects | A list of plans for this service, schema is defined below. MUST contain at least one plan. |

\* Fields with an asterisk are REQUIRED.

Note: Platforms will typically use the service name as an input parameter
from their users to indicate which service they want to instantiate. Therefore,
it is important that these values be unique for all services that have been
registered with a Platform. To achieve this goal service providers often will
prefix their service names with some unique value (such as the name of their
company). Additionally, some Platforms might modify the service names before
presenting them to their users. This specification places no requirements on
how Platforms might expose these values to their users.

##### Plan Object

| Response Field | Type | Description |
| --- | --- | --- |
| id* | string | An identifier used to correlate this plan in future requests to the Service Broker. This MUST be globally unique such that Platforms (and their users) MUST be able to assume that seeing the same value (no matter what Service Broker uses it) will always refer to this plan and for the same service. MUST be a non-empty string. Using a GUID is RECOMMENDED. |
| name* | string | The CLI-friendly name of the plan. MUST only contain alphanumeric characters, periods, and hyphens (no spaces). MUST be unique within the service. MUST be a non-empty string. |
| description* | string | A short description of the plan. MUST be a non-empty string. |
| metadata | object | An opaque object of metadata for a Service Plan. It is expected that Platforms will treat this as a blob. Note that there are [conventions](profile.md#service-metadata) in existing Service Brokers and Platforms for fields that aid in the display of catalog data. |
| free | boolean | When false, Service Instances of this plan have a cost. The default is true. |
| bindable | boolean | Specifies whether Service Instances of the Service Plan can be bound to applications. This field is OPTIONAL. If specified, this takes precedence over the `bindable` attribute of the service. If not specified, the default is derived from the service. |
| schemas | [Schemas](#schemas-object) | Schema definitions for Service Instances and Service Bindings for the plan. |

\* Fields with an asterisk are REQUIRED.

##### Schemas Object

| Response Field | Type | Description |
| --- | --- | --- |
| service_instance | [ServiceInstanceSchema](#service-instance-schema-object) | The schema definitions for creating and updating a Service Instance. |
| service_binding | [ServiceBindingSchema](#service-binding-schema-object) | The schema definition for creating a Service Binding. Used only if the Service Plan is bindable. |

##### Service Instance Schema Object

| Response Field | Type | Description |
| --- | --- | --- |
| create | [InputParametersSchema](#input-parameters-schema-object) | The schema definition for creating a Service Instance. |
| update | [InputParametersSchema](#input-parameters-schema-object) | The schema definition for updating a Service Instance. |

##### Service Binding Schema Object

| Response Field | Type | Description |
| --- | --- | --- |
| create | [InputParametersSchema](#input-parameters-schema-object) | The schema definition for creating a Service Binding. |

##### Input Parameters Schema Object

| Response Field | Type | Description |
| --- | --- | --- |
| parameters | JSON schema object | The schema definition for the input parameters. Each input parameter is expressed as a property within a JSON object. |

The following rules apply if `parameters` is included anywhere in the catalog:
* Platforms MUST support at least
[JSON Schema draft v4](http://json-schema.org/).
* Platforms SHOULD be prepared to support later versions of JSON schema.
* The `$schema` key MUST be present in the schema declaring the version of JSON
schema being used.
* Schemas MUST NOT contain any external references.
* Schemas MUST NOT be larger than 64kB.

```
{
  "services": [{
    "name": "fake-service",
    "id": "acb56d7c-XXXX-XXXX-XXXX-feb140a59a66",
    "description": "A fake service.",
    "tags": ["no-sql", "relational"],
    "requires": ["route_forwarding"],
    "bindable": true,
    "instances_retrievable": true,
    "bindings_retrievable": true,
    "metadata": {
      "provider": {
        "name": "The name"
      },
      "listing": {
        "imageUrl": "http://example.com/cat.gif",
        "blurb": "Add a blurb here",
        "longDescription": "A long time ago, in a galaxy far far away..."
      },
      "displayName": "The Fake Service Broker"
    },
    "plan_updateable": true,
    "plans": [{
      "name": "fake-plan-1",
      "id": "d3031751-XXXX-XXXX-XXXX-a42377d3320e",
      "description": "Shared fake Server, 5tb persistent disk, 40 max concurrent connections.",
      "free": false,
      "metadata": {
        "max_storage_tb": 5,
        "costs":[
            {
               "amount":{
                  "usd":99.0
               },
               "unit":"MONTHLY"
            },
            {
               "amount":{
                  "usd":0.99
               },
               "unit":"1GB of messages over 20GB"
            }
         ],
        "bullets": [
          "Shared fake server",
          "5 TB storage",
          "40 concurrent connections"
        ]
      },
      "schemas": {
        "service_instance": {
          "create": {
            "parameters": {
              "$schema": "http://json-schema.org/draft-04/schema#",
              "type": "object",
              "properties": {
                "billing-account": {
                  "description": "Billing account number used to charge use of shared fake server.",
                  "type": "string"
                }
              }
            }
          },
          "update": {
            "parameters": {
              "$schema": "http://json-schema.org/draft-04/schema#",
              "type": "object",
              "properties": {
                "billing-account": {
                  "description": "Billing account number used to charge use of shared fake server.",
                  "type": "string"
                }
              }
            }
          }
        },
        "service_binding": {
          "create": {
            "parameters": {
              "$schema": "http://json-schema.org/draft-04/schema#",
              "type": "object",
              "properties": {
                "billing-account": {
                  "description": "Billing account number used to charge use of shared fake server.",
                  "type": "string"
                }
              }
            }
          }
        }
      }
    }, {
      "name": "fake-plan-2",
      "id": "0f4008b5-XXXX-XXXX-XXXX-dace631cd648",
      "description": "Shared fake Server, 5tb persistent disk, 40 max concurrent connections. 100 async.",
      "free": false,
      "metadata": {
        "max_storage_tb": 5,
        "costs":[
            {
               "amount":{
                  "usd":199.0
               },
               "unit":"MONTHLY"
            },
            {
               "amount":{
                  "usd":0.99
               },
               "unit":"1GB of messages over 20GB"
            }
         ],
        "bullets": [
          "40 concurrent connections"
        ]
      }
    }]
  }]
}
```


### Adding a Service Broker to the Platform

After implementing the first endpoint `GET /v2/catalog` documented
[above](#catalog-management), the Service Broker will need to be registered
with your Platform to make your services and plans available to end users.

## Synchronous and Asynchronous Operations

Platforms expect prompt responses to all API requests in order to provide
users with fast feedback. Service Broker authors SHOULD implement their
Service Brokers to respond promptly to all requests but will need to decide
whether to implement synchronous or asynchronous responses. Service Brokers
that can guarantee completion of the requested operation with the response
SHOULD return the synchronous response. Service Brokers that cannot guarantee
completion of the operation with the response SHOULD implement the
asynchronous response.

Providing a synchronous response for a provision, update, or bind operation
before actual completion causes confusion for users as their service might not
be usable and they have no way to find out when it will be. Asynchronous
responses set expectations for users that an operation is in progress and can
also provide updates on the status of the operation.

Support for synchronous or asynchronous responses MAY vary by Service
Offering, even by Service Plan.

### Synchronous Operations

To execute a request synchronously, the Service Broker need only return the
usual status codes: `201 Created` for provision and bind, and `200 OK` for
update, unbind, and deprovision.

Service Brokers that support synchronous responses for provision, update, and
delete can ignore the `accepts_incomplete=true` query parameter if it is
provided by the client.

### Asynchronous Operations

For a Service Broker to return an asynchronous response, the query parameter
`accepts_incomplete=true` MUST be included the request. If the parameter is not
included or is set to `false`, and the Service Broker cannot fulfil the request
synchronously (guaranteeing that the operation is complete on response), then
the Service Broker MUST reject the request with the status code `422
Unprocessable Entity` and a response body containing error code
`"AsyncRequired"` (see [Service Broker Errors](#service-broker-errors)). The
error response MAY include a helpful error message in the `description` field
such as `"This Service Plan requires client support for asynchronous service
operations."`.

If the query parameter described above is present, and the Service Broker
executes the request asynchronously, the Service Broker MUST return the
asynchronous response `202 Accepted`.

An asynchronous response triggers the Platform to poll the Service Instance
or Service Binding's `last_operation` endpoint until the Service Broker
indicates that the requested operation has succeeded or failed. Service Brokers
MAY include a status message with each response for the `last_operation`
endpoint that provides visibility to end users as to the progress of the
operation.

## Blocking Operations

Service Brokers do not have to support concurrent requests that mutate the same
resource. If a Service Broker receives a request that it is not able to process
due to other activity being done on that resource then the Service Broker MUST
reject the request with a HTTP `422 Unprocessable Entity` and a response body
containing error code `"ConcurrencyError"` (see
[Service Broker Errors](#service-broker-errors)). The error response MAY include
a helpful error message in the `description` field such as `"Another operation
for this Service Instance is in progress."`.

Note that per the [Orphans](#orphans) section, this error response does not
cause orphan mitigation to be initiated. Therefore, Platforms receiving this
error response SHOULD resend the request at a later time.

Brokers MAY choose to treat the creation of a Service Binding as a mutation of
the corresponding Service Instance - it is an implementation choice. Doing so
would cause Platforms to serialize multiple Service Binding creation requests
when they are directed at the same Service Instance if concurrent updates are
not supported.

## Polling Last Operation for Service Instances

When a Service Broker returns status code `202 Accepted` for
[Provision](#provisioning), [Update](#updating-a-service-instance), or
[Deprovision](#deprovisioning), the Platform will begin polling the
`/v2/service_instances/:instance_id/last_operation` endpoint to obtain the
state of the last requested operation.

Returning `"state": "succeeded"` or `"state": "failed"` will cause the Platform
to cease polling.

### Request

#### Route
`GET /v2/service_instances/:instance_id/last_operation`

`:instance_id` MUST be the ID of a previously provisioned Service Instance.

#### Parameters

| Query-String Field | Type | Description |
| --- | --- | --- |
| service_id | string | If present, it MUST be the ID of the service being used. |
| plan_id | string | If present, it MUST be the ID of the plan for the Service Instance. If this endpoint is being polled as a result of changing the plan through a [Service Instance Update](#updating-a-service-instance), the ID of the plan prior to the update MUST be used. |
| operation | string | A Service Broker-provided identifier for the operation. When a value for `operation` is included with asynchronous responses for [Provision](#provisioning), [Update](#updating-a-service-instance), and [Deprovision](#deprovisioning) requests, the Platform MUST provide the same value using this query parameter as a percent-encoded string. If present, MUST be a non-empty string. |

Note: Although the request query parameters `service_id` and `plan_id` are not
mandatory, the Platform SHOULD include them on all `last_operation` requests
it makes to Service Brokers.

#### cURL
```
$ curl http://username:password@service-broker-url/v2/service_instances/:instance_id/last_operation -H "X-Broker-API-Version: 2.14"
```

### Response

| Status Code | Description |
| --- | --- |
| 200 OK | MUST be returned upon successful processing of this request. The expected response body is below. |
| 400 Bad Request | MUST be returned if the request is malformed or missing mandatory data. |
| 410 Gone | Appropriate only for asynchronous delete operations. The Platform MUST consider this response a success and forget about the resource. Returning this while the Platform is polling for create or update operations SHOULD be interpreted as an invalid response and the Platform SHOULD continue polling. |

Responses with any other status code SHOULD be interpreted as an error or
invalid response. The Platform SHOULD continue polling until the Service Broker
returns a valid response or the
[maximum polling duration](#polling-interval-and-duration) is reached.

#### Body

For success responses, the following fields are defined:

| Response Field | Type | Description |
| --- | --- | --- |
| state* | string | Valid values are `in progress`, `succeeded`, and `failed`. While `"state": "in progress"`, the Platform SHOULD continue polling. A response with `"state": "succeeded"` or `"state": "failed"` MUST cause the Platform to cease polling. |
| description | string | A user-facing message that can be used to tell the user details about the status of the operation. If present, MUST be a non-empty string. |

\* Fields with an asterisk are REQUIRED.

```
{
  "state": "in progress",
  "description": "Creating service (10% complete)."
}
```

If the response contains `"state": "failed"` then the Platform MUST send a
deprovision request to the Service Broker to prevent an orphan being created on
the Service Broker. However, while the Platform will attempt
to send a deprovision request, Service Brokers MAY automatically delete
any resources associated with the failed provisioning request on their own.

## Polling Last Operation for Service Bindings

When a broker returns status code `202 Accepted` for [Binding](#binding) or
[Unbinding](#unbinding), the Platform will begin polling the
`/v2/service_instances/:instance_id/service_bindings/:binding_id/last_operation`
endpoint to obtain the state of the last requested operation.

Returning `"state": "succeeded"` or `"state": "failed"` will cause the Platform
to cease polling and, in the case of a [Binding](#binding) request, information
regarding the Service Binding can then immediately be fetched using the
[Fetching a Service Binding](#fetching-a-service-binding) endpoint.

### Request

#### Route
`GET /v2/service_instances/:instance_id/service_bindings/:binding_id/last_operation`

`:instance_id` MUST be the ID of a previously provisioned Service Instance.

`:binding_id` MUST be the ID of a previously provisioned Service Binding for that
instance.

#### Parameters

The request provides these query string parameters as useful hints for brokers.

| Query-String Field | Type | Description |
| --- | --- | --- |
| service_id | string | ID of the service from the catalog. If present, MUST be a non-empty string. |
| plan_id | string | ID of the plan from the catalog. If present, MUST be a non-empty string. |
| operation | string | A broker-provided identifier for the operation. When a value for `operation` is included with asynchronous responses for [Binding](#binding) and [Unbinding](#unbinding) requests, the Platform MUST provide the same value using this query parameter as a URL-encoded string. If brokers do not return this `operation` field, only one asynchronous operation MAY be supported at a time. If present, MUST be a non-empty string. |

Note: Although the request query parameters `service_id` and `plan_id` are not
mandatory, the Platform SHOULD include them on all `last_operation` requests it
makes to Service Brokers.

#### cURL
```
$ curl http://username:password@broker-url/v2/service_instances/:instance_id/service_bindings/:binding_id/last_operation
```

### Response

| Status Code | Description |
| --- | --- |
| 200 OK | MUST be returned upon successful processing of this request. The expected response body is below. |
| 400 Bad Request | MUST be returned if the request is malformed or missing mandatory data. |
| 410 Gone | Appropriate only for asynchronous delete operations. The Platform MUST consider this response a success and remove the resource from its database. Returning this while the Platform is polling for create operations SHOULD be interpreted as an invalid response and the Platform SHOULD continue polling. |

Responses with any other status code SHOULD be interpreted as an error or
invalid response. The Platform SHOULD continue polling until the broker returns
a valid response or the
[maximum polling duration](#polling-interval-and-duration) is reached.

#### Body

For success responses, the following fields are defined:

| Response field | Type | Description |
| --- | --- | --- |
| state* | string | Valid values are `in progress`, `succeeded`, and `failed`. While `"state": "in progress"`, the Platform SHOULD continue polling. A response with `"state": "succeeded"` or `"state": "failed"` MUST cause the Platform to cease polling. |
| description | string | A user-facing message that can be used to tell the user details about the status of the operation. If present, MUST be a non-empty string. |

\* Fields with an asterisk are REQUIRED.

```
{
  "state": "in progress",
  "description": "Creating binding (10% complete)."
}
```

If the response contains `"state": "failed"` then the Platform MUST send an
unbind request to the Service Broker to prevent an orphan being created on
the Service Broker.

## Polling Interval and Duration

The frequency and maximum duration of polling MAY vary by Platform client. If
a Platform has a max polling duration and this limit is reached, the Platform
MUST cease polling and the operation state MUST be considered `failed`.

## Provisioning

When the Service Broker receives a provision request from the Platform, it
MUST take whatever action is necessary to create a new resource. What
provisioning represents varies by service and plan, although there are several
common use cases. For a MySQL service, provisioning could result in an empty
dedicated database server running on its own VM or an empty schema on a shared
database server. For non-data services, provisioning could just mean an
account on an multi-tenant SaaS application.

### Request

#### Route
`PUT /v2/service_instances/:instance_id`

`:instance_id` MUST be a globally unique non-empty string.
This ID will be used for future requests (bind and deprovision), so the
Service Broker will use it to correlate the resource it creates.

#### Parameters
| Parameter Name | Type | Description |
| --- | --- | --- |
| accepts_incomplete | boolean | A value of true indicates that the Platform and its clients support asynchronous Service Broker operations. If this parameter is not included in the request, and the Service Broker can only provision a Service Instance of the requested plan asynchronously, the Service Broker MUST reject the request with a `422 Unprocessable Entity` as described below. |

#### Body
| Request Field | Type | Description |
| --- | --- | --- |
| service_id* | string | MUST be the ID of a service from the catalog for this Service Broker. |
| plan_id* | string | MUST be the ID of a plan from the service that has been requested. |
| context | object | Platform specific contextual information under which the Service Instance is to be provisioned. Although most Service Brokers will not use this field, it could be helpful in determining data placement or applying custom business rules. `context` will replace `organization_guid` and `space_guid` in future versions of the specification - in the interim both SHOULD be used to ensure interoperability with old and new implementations. |
| organization_guid* | string | Deprecated in favor of `context`. The Platform GUID for the organization under which the Service Instance is to be provisioned. Although most Service Brokers will not use this field, it might be helpful for executing operations on a user's behalf. MUST be a non-empty string. |
| space_guid* | string | Deprecated in favor of `context`. The identifier for the project space within the Platform organization. Although most Service Brokers will not use this field, it might be helpful for executing operations on a user's behalf. MUST be a non-empty string. |
| parameters | object | Configuration parameters for the Service Instance. Service Brokers SHOULD ensure that the client has provided valid configuration parameters and values for the operation. |

\* Fields with an asterisk are REQUIRED.

```
{
  "service_id": "service-id-here",
  "plan_id": "plan-id-here",
  "context": {
    "platform": "cloudfoundry",
    "some_field": "some-contextual-data"
  },
  "organization_guid": "org-guid-here",
  "space_guid": "space-guid-here",
  "parameters": {
    "parameter1": 1,
    "parameter2": "foo"
  }
}
```

#### cURL
```
$ curl http://username:password@service-broker-url/v2/service_instances/:instance_id?accepts_incomplete=true -d '{
  "service_id": "service-id-here",
  "plan_id": "plan-id-here",
  "context": {
    "platform": "cloudfoundry",
    "some_field": "some-contextual-data"
  },
  "organization_guid": "org-guid-here",
  "space_guid": "space-guid-here",
  "parameters": {
    "parameter1": 1,
    "parameter2": "foo"
  }
}' -X PUT -H "X-Broker-API-Version: 2.14" -H "Content-Type: application/json"
```

### Response

| Status Code | Description |
| --- | --- |
| 200 OK | SHOULD be returned if the Service Instance already exists, is fully provisioned, and the requested parameters are identical to the existing Service Instance. The expected response body is below. |
| 201 Created | MUST be returned if the Service Instance was provisioned as a result of this request. The expected response body is below. |
| 202 Accepted | MUST be returned if the Service Instance provisioning is in progress. The `operation` string MUST match that returned for the original request. This triggers the Platform to poll the [Last Operation for Service Instances](#polling-last-operation-for-service-instances) endpoint for operation status. Note that a re-sent `PUT` request MUST return a `202 Accepted`, not a `200 OK`, if the Service Instance is not yet fully provisioned. |
| 400 Bad Request | MUST be returned if the request is malformed or missing mandatory data. |
| 409 Conflict | MUST be returned if a Service Instance with the same id already exists but with different attributes. |
| 422 Unprocessable Entity | MUST be returned if the Service Broker only supports asynchronous provisioning for the requested plan and the request did not include `?accepts_incomplete=true`. The response body MUST contain error code `"AsyncRequired"` (see [Service Broker Errors](#service-broker-errors)). The error response MAY include a helpful error message in the `description` field such as `"This Service Plan requires client support for asynchronous service operations."`. |

Responses with any other status code MUST be interpreted as a failure. See
the [Orphans](#orphans) section for more information related to whether
orphan mitigation needs to be applied. While a Platform might attempt
to send a deprovision request, Service Brokers MAY automatically delete
any resources associated with the failed provisioning request on their own.

#### Body

For success responses, the following fields are defined:

| Response Field | Type | Description |
| --- | --- | --- |
| dashboard_url | string | The URL of a web-based management user interface for the Service Instance; we refer to this as a service dashboard. The URL MUST contain enough information for the dashboard to identify the resource being accessed (`9189kdfsk0vfnku` in the example below). Note: a Service Broker that wishes to return `dashboard_url` for a Service Instance MUST return it with the initial response to the provision request, even if the service is provisioned asynchronously. If present, MUST be a non-empty string. |
| operation | string | For asynchronous responses, Service Brokers MAY return an identifier representing the operation. The value of this field MUST be provided by the Platform with requests to the [Polling Last Operation for Service Instances](#polling-last-operation-for-service-instances) endpoint in a percent-encoded query parameter. If present, MUST be a non-empty string. |

```
{
  "dashboard_url": "http://example-dashboard.example.com/9189kdfsk0vfnku",
  "operation": "task_10"
}
```

## Fetching a Service Instance

If `"instances_retrievable" :true` is declared for a service in the
[Catalog](#catalog-management) endpoint, Service Brokers MUST support this
endpoint for all plans of the service and this endpoint MUST be available
immediately after the
[Polling Last Operation for Service Instances](#polling-last-operation-for-service-instances)
endpoint returns `"state": "succeeded"` for a [Provisioning](#provisioning)
operation. Otherwise, Platforms SHOULD NOT attempt to call this endpoint under
any circumstances.

### Request

##### Route
`GET /v2/service_instances/:instance_id`

`:instance_id` MUST be the ID of a previously provisioned Service Instance.

##### cURL
```
$ curl 'http://username:password@broker-url/v2/service_instances/:instance_id' -X GET -H "X-Broker-API-Version: 2.14"
```

### Response

| Status Code | Description |
| --- | --- |
| 200 OK | The expected response body is below. |
| 404 Not Found | MUST be returned if the Service Instance does not exist or if a provisioning operation is still in progress. |
| 422 Unprocessable Entity | MUST be returned if the Service Instance is being updated and therefore cannot be fetched at this time. The response body MUST contain error code `"ConcurrencyError"` (see [Service Broker Errors](#service-broker-errors)). |

Responses with any other status code MUST be interpreted as a failure and the
Platform MUST continue to remember the Service Instance.

##### Body

For success responses, the following fields are defined:

| Response field | Type | Description |
| --- | --- | --- |
| service_id | string | The ID of the service from the catalog that is associated with the Service Instance. |
| plan_id | string | The ID of the plan from the catalog that is associated with the Service Instance. |
| dashboard_url | string | The URL of a web-based management user interface for the Service Instance; we refer to this as a service dashboard. The URL MUST contain enough information for the dashboard to identify the resource being accessed (`9189kdfsk0vfnku` in the example below). Note: a Service Broker that wishes to return `dashboard_url` for a Service Instance MUST return it with the initial response to the provision request, even if the service is provisioned asynchronously. |
| parameters | object | Configuration parameters for the Service Instance. |

Service Brokers MAY choose to not return some or all parameters when a Service Instance is fetched - for example,
if it contains sensitive information.

```
{
  "dashboard_url": "http://example-dashboard.example.com/9189kdfsk0vfnku",
  "parameters": {
    "billing-account": "abcde12345"
  }
}
```

## Updating a Service Instance

By implementing this endpoint, Service Broker authors can enable users to
modify two attributes of an existing Service Instance: the Service Plan and
parameters. By changing the Service Plan, users can upgrade or downgrade their
Service Instance to other plans. By modifying parameters, users can change
configuration options that are specific to a service or plan.

To enable support for the update of the plan, a Service Broker MUST declare
support per service by including `"plan_updateable": true` in its [catalog
endpoint](#catalog-management).

If `"plan_updateable": true` is declared for a plan in the
[Catalog](#catalog-management) endpoint, the Platform MAY request a plan change
on a Service Instance using the given plan. Otherwise, Platforms MUST NOT make
any plan change requests to the Service Broker for any Service Instance using
the given plan, but MAY request an update to the Service Instance parameters.

Not all permutations of plan changes are expected to be supported. For
example, a service might support upgrading from plan "shared small" to "shared
large" but not to plan "dedicated". It is up to the Service Broker to validate
whether a particular permutation of plan change is supported. If a particular
plan change is not supported, the Service Broker SHOULD return a meaningful
error message in response.

### Request

#### Route
`PATCH /v2/service_instances/:instance_id`

`:instance_id` MUST be the ID of a previously provisioned Service Instance.

#### Parameters
| Parameter Name | Type | Description |
| --- | --- | --- |
| accepts_incomplete | boolean | A value of true indicates that the Platform and its clients support asynchronous Service Broker operations. If this parameter is not included in the request, and the Service Broker can only update a Service Instance of the requested plan asynchronously, the Service Broker MUST reject the request with a `422 Unprocessable Entity` as described below. |

#### Body

| Request Field | Type | Description |
| --- | --- | --- |
| context | object | Contextual data under which the Service Instance is created. |
| service_id* | string | MUST be the ID of a service from the catalog for this Service Broker. |
| plan_id | string | If present, MUST be the ID of a plan from the service that has been requested. If this field is not present in the request message, then the Service Broker MUST NOT change the plan of the Service Instance as a result of this request. |
| parameters | object | Configuration parameters for the Service Instance. Service Brokers SHOULD ensure that the client has provided valid configuration parameters and values for the operation. See "Note" below. |
| previous_values | [PreviousValues](#previous-values-object) | Information about the Service Instance prior to the update. |

\* Fields with an asterisk are REQUIRED.

##### Previous Values Object

| Request Field | Type | Description |
| --- | --- | --- |
| service_id | string | Deprecated; determined to be unnecessary as the value is immutable. If present, it MUST be the ID of the service for the Service Instance. |
| plan_id | string | If present, it MUST be the ID of the plan prior to the update. |
| organization_id | string | Deprecated as it was redundant information. Organization for the Service Instance MUST be provided by Platforms in the top-level field `context`. If present, it MUST be the ID of the organization specified for the Service Instance. |
| space_id | string | Deprecated as it was redundant information. Space for the Service Instance MUST be provided by Platforms in the top-level field `context`. If present, it MUST be the ID of the space specified for the Service Instance. |

Note: The `parameters` specified are expected to be the values specified
by an end-user. Whether the user chooses to include the complete set of
configuration options or just a subset (or even none) is their choice. How a
Service Broker interprets these parameters (including the absence of any
particular parameter) is out of scope of this specification - with the
exception that if this field is not present in the request then the
Service Broker MUST NOT change the parameters of the instance as a result of
this request.

Since some Service Instances will provide a `dashboard_url`, it is possible
that a user has modified some of these parameters via the dashboard and
therefore the Platform might not be aware of these changes. For this reason,
Platforms SHOULD NOT include any parameters on the request that
the user did not explicitly specify in their request for the update.

\* Fields with an asterisk are REQUIRED.

```
{
  "context": {
    "platform": "cloudfoundry",
    "some_field": "some-contextual-data"
  },
  "service_id": "service-id-here",
  "plan_id": "plan-id-here",
  "parameters": {
    "parameter1": 1,
    "parameter2": "foo"
  },
  "previous_values": {
    "plan_id": "old-plan-id-here",
    "service_id": "service-id-here",
    "organization_id": "org-guid-here",
    "space_id": "space-guid-here"
  }
}
```

#### cURL
```
$ curl http://username:password@service-broker-url/v2/service_instances/:instance_id?accepts_incomplete=true -d '{
  "context": {
    "platform": "cloudfoundry",
    "some_field": "some-contextual-data"
  },
  "service_id": "service-id-here",
  "plan_id": "plan-id-here",
  "parameters": {
    "parameter1": 1,
    "parameter2": "foo"
  },
  "previous_values": {
    "plan_id": "old-plan-id-here",
    "service_id": "service-id-here",
    "organization_id": "org-guid-here",
    "space_id": "space-guid-here"
  }
}' -X PATCH -H "X-Broker-API-Version: 2.14" -H "Content-Type: application/json"
```

### Response

| Status Code | Description |
| --- | --- |
| 200 OK | MUST be returned if the request's changes have been applied or MAY be returned if the request's changes have had no effect. The expected response body is `{}`. |
| 202 Accepted | MUST be returned if the Service Instance update is in progress. The `operation` string MUST match that returned for the original request. This triggers the Platform to poll the [Polling Last Operation for Service Instances](#polling-last-operation-for-service-instances) endpoint for operation status. Note that a re-sent `PATCH` request MUST return a `202 Accepted`, not a `200 OK`, if the requested update has not yet completed. |
| 400 Bad Request | MUST be returned if the request is malformed or missing mandatory data. |
| 422 Unprocessable entity | MUST be returned if the requested change is not supported or if the request cannot currently be fulfilled due to the state of the Service Instance (e.g. Service Instance utilization is over the quota of the requested plan). Additionally, a `422 Unprocessable Entity` MUST be returned if the Service Broker only supports asynchronous update for the requested plan and the request did not include `?accepts_incomplete=true`; in this case the response body MUST contain a error code `"AsyncRequired"` (see [Service Broker Errors](#service-broker-errors)). The error response MAY include a helpful error message in the `description` field such as `"This Service Plan requires client support for asynchronous service operations."`. |

Responses with any other status code MUST be interpreted as a failure.
When the response includes a 4xx status code, the Service Broker MUST NOT
apply any of the requested changes to the Service Instance.

#### Body

For success responses, the following fields are defined:

| Response Field | Type | Description |
| --- | --- | --- |
| dashboard_url | string | The updated URL of a web-based management user interface for the Service Instance; we refer to this as a service dashboard. The URL MUST contain enough information for the dashboard to identify the resource being accessed (`0129d920a083838` in the example below). Note: a Service Broker that wishes to return `dashboard_url` for a Service Instance MUST return it with the initial response to the update request, even if the Service Instance is being updated asynchronously. If present, MUST be a non-empty string. If not present, the Platform MUST retain the previous value of the `dashboard_url`. |
| operation | string | For asynchronous responses, Service Brokers MAY return an identifier representing the operation. The value of this field MUST be provided by the Platform with requests to the [Polling Last Operation for Service Instances](#polling-last-operation-for-service-instances) endpoint in a percent-encoded query parameter. If present, MUST be a non-empty string. |

```
{
  "dashboard_url": "http://example-dashboard.example.com/0129d920a083838",
  "operation": "task_10"
}
```


## Binding

If `"bindable": true` is declared for a service or plan in the
[Catalog](#catalog-management) endpoint, the Platform MAY request generation
of a Service Binding. Otherwise, Platforms MUST NOT make a binding request to
the Service Broker for any Service Instance using the given service or plan.

Note: Not all services need to be bindable --- some deliver value just from
being provisioned. Service Brokers that offer services that are bindable MUST
declare them as such using `"bindable": true` in the
[Catalog](#catalog-management). Service Brokers that do not offer any bindable
services do not need to implement the endpoint for bind requests.

Service Brokers MAY choose to only return the information that represents a
Service Binding once, either when the Service Binding is being created
synchronously, or when the Service Binding is first fetched via the [Fetching a
Service Binding](#fetching-a-service-binding) endpoint. However, in order for
the Platform to successfully use the Service Binding, the information MUST be
returned at least once.

### Types of Binding

#### Credentials

Credentials are a set of information used by an Application or a user to
utilize the Service Instance. Credentials SHOULD be unique whenever possible, so
access can be revoked for each Service Binding without affecting consumers of other
Service Bindings for the Service Instance.

#### Log Drain

There are a class of Service Offerings that provide aggregation, indexing, and
analysis of log data. To utilize these services an application that generates
logs needs information for the location to which it will stream logs. A create
binding response from a Service Broker that provides one of these services MUST
include a `syslog_drain_url`. The Platform MUST use the `syslog_drain_url` value
when sending logs to the service.

#### Route Services
Route services are a class of Service Offerings that intermediate requests to
applications, performing functions such as rate limiting or authorization. To
indicate support for route services, the catalog entry for the Service MUST
include the `"requires":["route_forwarding"]` property.

When creating a route service type of Service Binding, a Platform MUST send
a routable address, or endpoint, for the application along with the request to
create a Service Binding using `"bind_resource":{"route":"some-address.com"}`.
Service Brokers MAY support configuration specific to an address using
parameters; exposing this feature to users would require a Platform to support
binding multiple routable addresses to the same Service Instance.

If a service is deployed in a configuration to support this behavior, the
Service Broker MUST return a `route_service_url` in the response for a request
to create a Service Binding, so that the Platform knows where to proxy the application
request. If the service is deployed such that the network configuration to
proxy application requests through instances of the service is managed
out-of-band, the Service Broker MUST NOT return `route_service_url` in the
response.

#### Volume Services

There are a class of services that provide network storage to applications
via volume mounts in the application container. A create Service Binding response from
one of these services MUST include `volume_mounts`.

### Request

#### Route
`PUT /v2/service_instances/:instance_id/service_bindings/:binding_id`

`:instance_id` MUST be the ID of a previously provisioned Service Instance.

`:binding_id` MUST be a globally unique non-empty string.
This ID will be used for future unbind requests, so the Service Broker will use
it to correlate the resource it creates.

#### Parameters
| Parameter name | Type | Description |
| --- | --- | --- |
| accepts_incomplete | boolean | A value of true indicates that the Platform and its clients support asynchronous broker operations. If this parameter is not included in the request, and the broker can only perform a binding operation asynchronously, the broker MUST reject the request with a `422 Unprocessable Entity` as described below. |

#### Body

| Request Field | Type | Description |
| --- | --- | --- |
| context | object | Contextual data under which the Service Binding is created. |
| service_id* | string | MUST be the ID of the service that is being used. |
| plan_id* | string | MUST be the ID of the plan from the service that is being used. |
| app_guid | string | Deprecated in favor of `bind_resource.app_guid`. GUID of an application associated with the Service Binding to be created. If present, MUST be a non-empty string. |
| bind_resource | [BindResource](#bind-resource-object) | A JSON object that contains data for Platform resources associated with the Service Binding to be created. See [Bind Resource Object](#bind-resource-object) for more information. |
| parameters | object | Configuration parameters for the Service Binding. Service Brokers SHOULD ensure that the client has provided valid configuration parameters and values for the operation. |

\* Fields with an asterisk are REQUIRED.

##### Bind Resource Object

The `bind_resource` object contains Platform specific information related to
the context in which the service will be used. In some cases the Platform
might not be able to provide this information at the time of the binding
request, therefore the `bind_resource` and its fields are OPTIONAL.

Below are some common fields that MAY be used. Platforms MAY choose to add
additional ones as needed (see
[Bind Resource Object](profile.md#bind-resource-object) conventions).

| Request Field | Type | Description |
| --- | --- | --- |
| app_guid | string | GUID of an application associated with the Service Binding. For [credentials](#types-of-binding) bindings. MUST be unique within the scope of the Platform. |
| route | string | URL of the application to be intermediated. For [route services](#route-services) Service Bindings. |

`app_guid` represents the scope to which the Service Binding will apply within
the Platform. For example, in Cloud Foundry it might map to an "application"
while in Kubernetes it might map to a "namespace". The scope of what a
Platform maps the `app_guid` to is Platform specific and MAY vary across
binding requests.

```
{
  "context": {
    "platform": "cloudfoundry",
    "some_field": "some-contextual-data"
  },
  "service_id": "service-id-here",
  "plan_id": "plan-id-here",
  "bind_resource": {
    "app_guid": "app-guid-here"
  },
  "parameters": {
    "parameter1-name-here": 1,
    "parameter2-name-here": "parameter2-value-here"
  }
}
```


#### cURL
```
$ curl http://username:password@service-broker-url/v2/service_instances/:instance_id/service_bindings/:binding_id?accepts_incomplete=true -d '{
  "context": {
    "platform": "cloudfoundry",
    "some_field": "some-contextual-data"
  },
  "service_id": "service-id-here",
  "plan_id": "plan-id-here",
  "bind_resource": {
    "app_guid": "app-guid-here"
  },
  "parameters": {
    "parameter1-name-here": 1,
    "parameter2-name-here": "parameter2-value-here"
  }
}' -X PUT -H "X-Broker-API-Version: 2.14" -H "Content-Type: application/json"
```

### Response

| Status Code | Description |
| --- | --- |
| 200 OK | SHOULD be returned if the Service Binding already exists and the requested parameters are identical to the existing Service Binding. The expected response body is below. |
| 201 Created | MUST be returned if the Service Binding was created as a result of this request. The expected response body is below. |
| 202 Accepted | MUST be returned if the binding is in progress. The `operation` string MUST match that returned for the original request. This triggers the Platform to poll the [Polling Last Operation for Service Bindings](#polling-last-operation-for-service-bindings) endpoint for operation status. Information regarding the Service Binding (i.e. credentials) MUST NOT be returned in this response. Note that a re-sent `PUT` request MUST return a `202 Accepted`, not a `200 OK`, if the Service Binding is not yet fully created. |
| 400 Bad Request | MUST be returned if the request is malformed or missing mandatory data. |
| 409 Conflict | MUST be returned if a Service Binding with the same id, for the same Service Instance, already exists but with different parameters. |
| 422 Unprocessable Entity | MUST be returned if the Service Broker requires that `app_guid` be included in the request body. The response body MUST contain error code `"RequiresApp"` (see [Service Broker Errors](#service-broker-errors)). The error response MAY include a helpful error message in the `description` field such as `"This Service supports generation of credentials through binding an application only."`. Additionally, if the Service Broker rejects the request due to a concurrent request to create a Service Binding for the same Service Instance, then this error MUST be returned (see [Blocking Operations](#blocking-operations)). This MUST also be returned if the Service Broker only supports asynchronous bindings for the Service Instance and the request did not include `?accepts_incomplete=true`. In this case, the response body MUST contain error code `"AsyncRequired"` (see [Service Broker Errors](#service-broker-errors)). The error response MAY include a helpful error message in the `description` field such as `"This Service Instance requires client support for asynchronous binding operations."`. |

Responses with any other status code MUST be interpreted as a failure and an
unbind request MUST be sent to the Service Broker to prevent an orphan being
created on the Service Broker. However, while the platform will attempt
to send an unbind request, Service Brokers MAY automatically delete
any resources associated with the failed bind request on their own.

#### Body

For a `202 Accepted` response code, the following fields are defined:

| Response Field | Type | Description |
| --- | --- | --- |
| operation | string | For asynchronous responses, Service Brokers MAY return an identifier representing the operation. The value of this field MUST be provided by the Platform with requests to the [Polling Last Operation for Service Bindings](#polling-last-operation-for-service-bindings) endpoint in a URL encoded query parameter. If present, MUST be a non-empty string. |

For `200 OK` and `201 Created` response codes, the following fields are defined:

| Response Field | Type | Description |
| --- | --- | --- |
| credentials | object | A free-form hash of credentials that can be used by applications or users to access the service. MUST be returned if the Service Broker supports generation of credentials. |
| syslog_drain_url | string | A URL to which logs MUST be streamed. `"requires":["syslog_drain"]` MUST be declared in the [Catalog](#catalog-management) endpoint or the Platform can consider the response invalid. |
| route_service_url | string | A URL to which the Platform MUST proxy requests for the address sent with `bind_resource.route` in the request body. `"requires":["route_forwarding"]` MUST be declared in the [Catalog](#catalog-management) endpoint or the Platform can consider the response invalid. |
| volume_mounts | array of [VolumeMount](#volume-mount-object) objects | An array of configuration for remote storage devices to be mounted into an application container filesystem. `"requires":["volume_mount"]` MUST be declared in the [Catalog](#catalog-management) endpoint or the Platform can consider the response invalid. |

##### Volume Mount Object

| Response Field | Type | Description |
| --- | --- | --- |
| driver* | string | Name of the volume driver plugin which manages the device. |
| container_dir* | string | The path in the application container onto which the volume will be mounted. This specification does not mandate what action the Platform is to take if the path specified already exists in the container. |
| mode* | string | "r" to mount the volume read-only or "rw" to mount it read-write. |
| device_type* | string | A string specifying the type of device to mount. Currently the only supported value is "shared". |
| device* | [Device](#device-object) | Device object containing device_type specific details. Currently only shared devices are supported. |

\* Fields with an asterisk are REQUIRED.

##### Device Object

Currently only shared devices are supported; a distributed file system which
can be mounted on all app instances simultaneously.

| Field | Type | Description |
| --- | --- | --- |
| volume_id* | string | ID of the shared volume to mount on every app instance. |
| mount_config | object | Configuration object to be passed to the driver when the volume is mounted. |

\* Fields with an asterisk are REQUIRED.

```
{
  "credentials": {
    "uri": "mysql://mysqluser:pass@mysqlhost:3306/dbname",
    "username": "mysqluser",
    "password": "pass",
    "host": "mysqlhost",
    "port": 3306,
    "database": "dbname"
  }
}
```

```
{
  "volume_mounts": [{
    "driver": "cephdriver",
    "container_dir": "/data/images",
    "mode": "r",
    "device_type": "shared",
    "device": {
      "volume_id": "bc2c1eab-05b9-482d-b0cf-750ee07de311",
      "mount_config": {
        "key": "value"
      }
    }
  }]
}
```

## Fetching a Service Binding

If `"bindings_retrievable" :true` is declared for a service in the
[Catalog](#catalog-management) endpoint, Service Brokers MUST support this
endpoint for all services and plans that support bindings (`"bindable": true`)
and this endpoint MUST be available immediately after the
[Polling Last Operation for Service Bindings](#polling-last-operation-for-service-bindings)
endpoint returns `"state": "succeeded"` for a [Binding](#binding) operation.

### Request

##### Route
`GET /v2/service_instances/:instance_id/service_bindings/:binding_id`

`:instance_id` MUST be the ID of a previously provisioned Service Instance.

`:binding_id` MUST be the ID of a previously provisioned Service Binding for that
instance.

##### cURL
```
$ curl 'http://username:password@broker-url/v2/service_instances/:instance_id/service_bindings/:binding_id' -X GET -H "X-Broker-API-Version: 2.14"
```

### Response

| Status Code | Description |
| --- | --- |
| 200 OK | The expected response body is below. |
| 404 Not Found | MUST be returned if the Service Binding does not exist or if a binding operation is still in progress. |

Responses with any other status code MUST be interpreted as a failure and the
Platform MUST continue to remember the Service Binding.

##### Body

For success responses, the following fields are defined:

| Response Field | Type | Description |
| --- | --- | --- |
| credentials | object | A free-form hash of credentials that can be used by applications or users to access the service. MUST be returned if the Service Broker supports generation of credentials and the Service Binding was provisioned asynchronously. |
| syslog_drain_url | string | A URL to which logs MUST be streamed. `"requires":["syslog_drain"]` MUST be declared in the [Catalog](#catalog-management) endpoint or the Platform can consider the response invalid. |
| route_service_url | string | A URL to which the Platform MUST proxy requests for the address sent with `bind_resource.route` in the request body. `"requires":["route_forwarding"]` MUST be declared in the [Catalog](#catalog-management) endpoint or the Platform can consider the response invalid. |
| volume_mounts | array of [VolumeMount](#volume-mount-object) objects | An array of configuration for mounting volumes. `"requires":["volume_mount"]` MUST be declared in the [Catalog](#catalog-management) endpoint or the Platform can consider the response invalid. |
| parameters | object | Configuration parameters for the Service Binding. |

Service Brokers MAY choose to not return some or all parameters when a Service Binding is fetched - for example,
if it contains sensitive information.

```
{
  "credentials": {
    "uri": "mysql://mysqluser:pass@mysqlhost:3306/dbname",
    "username": "mysqluser",
    "password": "pass",
    "host": "mysqlhost",
    "port": 3306,
    "database": "dbname"
  },
  "parameters": {
    "billing-account": "abcde12345"
  }
}
```

## Unbinding

Note: Service Brokers that do not provide any bindable services or plans do
not need to implement this endpoint.

When a Service Broker receives an unbind request from a Platform, it MUST
delete any resources associated with the Service Binding. In the case where
credentials were generated, this might result in requests to the Service
Instance failing to authenticate.

### Request

#### Route

`DELETE /v2/service_instances/:instance_id/service_bindings/:binding_id`

`:instance_id` MUST be the ID of a previously provisioned Service Instance.

`:binding_id` MUST be the the ID of a previously provisioned Service Binding for that
Service Instance.

#### Parameters

| Query-String Field | Type | Description |
| --- | --- | --- |
| service_id* | string | MUST be the ID of the service associated with the Service Binding being deleted. |
| plan_id* | string | MUST be the ID of the plan associated with the Service Binding being deleted. |
| accepts_incomplete | boolean | A value of true indicates that the Platform and its clients support asynchronous Service Broker operations. If this parameter is not included in the request, and the Service Broker can only perform an unbinding operation asynchronously, the Service Broker MUST reject the request with a `422 Unprocessable Entity` as described below. |

\* Query parameters with an asterisk are REQUIRED.

#### cURL

```
$ curl 'http://username:password@service-broker-url/v2/service_instances/:instance_id/
  service_bindings/:binding_id?service_id=service-id-here&plan_id=plan-id-here&accepts_incomplete=true' -X DELETE -H "X-Broker-API-Version: 2.14"
```

### Response

| Status Code | Description |
| --- | --- |
| 200 OK | MUST be returned if the Service Binding was deleted as a result of this request. The expected response body is `{}`. |
| 202 Accepted | MUST be returned if the unbinding is in progress. The `operation` string MUST match that returned for the original request. This triggers the Platform to poll the [Polling Last Operation for Service Bindings](#polling-last-operation-for-service-bindings) endpoint for operation status. Note that a re-sent `DELETE` request MUST return a `202 Accepted`, not a `200 OK`, if the unbinding request has not completed yet. |
| 400 Bad Request | MUST be returned if the request is malformed or missing mandatory data. |
| 410 Gone | MUST be returned if the Service Binding does not exist. |
| 422 Unprocessable Entity | MUST also be returned if the Service Broker only supports asynchronous unbinding for the Service Instance and the request did not include `?accepts_incomplete=true`. The response body MUST contain error code `"AsyncRequired"` (see [Service Broker Errors](#service-broker-errors)). The error response MAY include a helpful error message in the `description` field such as `"This Service Instance requires client support for asynchronous binding operations."`. |

Responses with any other status code MUST be interpreted as a failure and the
Platform MUST continue to remember the Service Binding.

#### Body

For success responses, the following fields are defined:

| Response field | Type | Description |
| --- | --- | --- |
| operation | string | For asynchronous responses, Service Brokers MAY return an identifier representing the operation. The value of this field MUST be provided by the Platform with requests to the [Polling Last Operation for Service Bindings](#polling-last-operation-for-service-bindings) endpoint in a percent-encoded query parameter. If present, MUST be a non-empty string. |

\* Fields with an asterisk are REQUIRED.

```
{
  "operation": "task_10"
}
```

## Deprovisioning

When a Service Broker receives a deprovision request from a Platform, it MUST
delete any resources it created during the provision. Usually this means that
all resources are immediately reclaimed for future provisions.

Platforms MUST delete all Service Bindings for a service prior to attempting to
deprovision the service. This specification does not specify what a Service
Broker is to do if it receives a deprovision request while there are still
Service Bindings associated with it.

### Request

#### Route

`DELETE /v2/service_instances/:instance_id`

`:instance_id` MUST be the ID of a previously provisioned Service Instance.

#### Parameters

The request provides these query string parameters as useful hints for Service
Brokers.


| Query-String Field | Type | Description |
| --- | --- | --- |
| service_id* | string | MUST be the ID of the service associated with the Service Instance being deleted. |
| plan_id* | string | MUST be the ID of the plan associated with the Service Instance being deleted. |
| accepts_incomplete | boolean | A value of true indicates that both the Platform and the requesting client support asynchronous deprovisioning. If this parameter is not included in the request, and the Service Broker can only deprovision a Service Instance of the requested plan asynchronously, the Service Broker MUST reject the request with a `422 Unprocessable Entity` as described below. |

\* Query parameters with an asterisk are REQUIRED.

#### cURL
```
$ curl 'http://username:password@service-broker-url/v2/service_instances/:instance_id?accepts_incomplete=true
  &service_id=service-id-here&plan_id=plan-id-here' -X DELETE -H "X-Broker-API-Version: 2.14"
```

### Response

| Status Code | Description |
| --- | --- |
| 200 OK | MUST be returned if the Service Instance was deleted as a result of this request. The expected response body is `{}`. |
| 202 Accepted | MUST be returned if the Service Instance deletion is in progress. The `operation` string MUST match that returned for the original request. This triggers the Platform to poll the [Polling Last Operation for Service Instances](#polling-last-operation-for-service-instances) endpoint for operation status. Note that a re-sent `DELETE` request MUST return a `202 Accepted`, not a `200 OK`, if the delete request has not completed yet. |
| 400 Bad Request | MUST be returned if the request is malformed or missing mandatory data. |
| 410 Gone | MUST be returned if the Service Instance does not exist. |
| 422 Unprocessable Entity | MUST be returned if the Service Broker only supports asynchronous deprovisioning for the requested plan and the request did not include `?accepts_incomplete=true`. The response body MUST contain error code `"AsyncRequired"` (see [Service Broker Errors](#service-broker-errors)). The error response MAY include a helpful error message in the `description` field such as `"This Service Plan requires client support for asynchronous service operations."`. |

Responses with any other status code MUST be interpreted as a failure and the
Platform MUST remember the Service Instance.

#### Body

For success responses, the following fields are defined:

| Response Field | Type | Description |
| --- | --- | --- |
| operation | string | For asynchronous responses, Service Brokers MAY return an identifier representing the operation. The value of this field MUST be provided by the Platform with requests to the [Polling Last Operation for Service Instances](#polling-last-operation-for-service-instances) endpoint in a percent-encoded query parameter. If present, MUST be a non-empty string. |

```
{
  "operation": "task_10"
}
```

## Orphans

The Platform is the source of truth for Service Instances and Service Bindings.
Service Brokers are expected to have successfully provisioned all of the Service
Instances and Service Bindings that the Platform knows about, and none that it
doesn't.

Orphans can result if the Service Broker does not return a response before a
request from the Platform times out (typically 60 seconds). For example, if a
Service Broker does not return a response to a provision request before the
request times out, the Service Broker might eventually succeed in provisioning
a Service Instance after the Platform considers the request a failure. This
results in an orphan Service Instance on the Service Broker's side.

To mitigate orphan Service Instances and Service Bindings, the Platform SHOULD
attempt to delete resources it cannot be sure were successfully created, and
SHOULD keep trying to delete them until the Service Broker responds with a
success.

Platforms SHOULD initiate orphan mitigation in the following scenarios:

| Status Code Of Service Broker Response | Platform Interpretation Of Response | Platform Initiates Orphan Mitigation? |
| --- | --- | --- |
| 200 | Success | No |
| 200 with malformed response | Failure | No |
| 201 | Success | No |
| 201 with malformed response | Failure | Yes |
| All other 2xx | Failure | Yes |
| 408 | Client timeout failure (request not received at the server) | No |
| All other 4xx | Request rejected | No |
| 5xx | Service Broker error | Yes |
| Timeout | Failure | Yes |

If the Platform encounters an internal error provisioning a Service Instance or
Service Binding (for example, saving to the database fails), then it MUST at
least send a single delete or unbind request to the Service Broker to prevent
the creation of an orphan.
