+++
title = "REST Standards"
description = "REST standards to follow while designing services"
categories = ["recipes"]
tags = ["REST", "Rest Standards"]
date = 2018-12-03T19:38:04-05:00
draft = false
+++

# REST Standards


## Purpose
This document defines URI standards, dos and don’ts for Unified API strategy. In FedEx RESTful APIs, the client specifies an
action using an HTTP verb such as POST, GET, PUT, or DELETE. It specifies a resource by a globally-unique URI of the following
[template](https://api.fedex.com/islandName/apiVersion/resourcePath/endpointPath?parameters)

![Hystrix Dashboard Screenshot 1](/images/rest-standards/rest_triangle.png)

## URI Standards
1. Use only Nouns (No Verbs)
2. Use plurals for resources like shipments/pickups
3. Http methods used for verb
 * GET: Retrieve resource
 * POST: Create resource
 * PUT: Update resource
 * DELETE: Delete resource
4. All the URI parts are in lower case
5. URI Hierarchy is important
6. Hackable 'up the tree'. The user should be able to remove the leaf path and get an expected page back.
7. Predictable> Human-guessable. If your URLs are meaningful they may also be predictable. If your users understand them and predict what a URL for a given resource is then may be able to go
'straight there' without having to find a hyperlink on a page. If your URIs are predictable, then your developers will argue less over what should be used for new resource types.
8. Query parameters are used for filtering/pagination/sorting.
9. Tied to a resource. Permanent. The URI will continue to work while the resource exists, and despite the resource potentially changing over the time. This basically represents versioning part of the resources which doesn't impacts or changes the current URI because every URI will have consumers.
10. Return a representation (e.g. XML or JSON) based on the request header, like Accept and Accept-Language rather than a change in the URI.

##### Things to Avoid
1. Do not use characters that require URL encoding in URIs (e.g. spaces)
2. Do not use singular nouns for resources
3. Do not use mixed or camel case in URIs
4. Do not use verb in URI. Few bad examples
 * POST https://api.fedex.com/update/shipment/12345
 * PUT https://api.fedex.com/update-shipment/12345
 * GET https://api.fedex.com/shipmentServices?action=updateshipment&trackingNumber=12345

**Examples**

1. https://api.fedex.com/ship/v1/shipments
 * GET: Retrieves list of shipments
 * POST: Create a new shipment
 * PUT: Not supported: Currently we are only support one shipment at a time.
 * DELETE: Not supported: Group shipment delete is not supported
2. https://api.fedex.com/ship/v1/shipments/{trackingNumber}
 * GET: Retrieves shipment details
 * POST: Not supported: Shipment is already created vhj
 * PUT: Update shipment details
 * DELETE: Delete shipment
3. https://api.fedex.com/country/v1/countries?type=sender
 * GET: Retrieves list of countries for the type sender
 * POST: Not supported
 * PUT: Not supported
 * DELETE: Not supported
4. https://api.fedex.com/country/v1/countries/US
 * GET: Retrieves the country detail for US
 * POST: Not supported
 * PUT: Not supported
 * DELETE: Not supported

## Resource Naming
Resource naming is most important concept when creating an understandable, easily leveraged Web service API.
When resources are named well, an API is intuitive and easy to use.
The constraint of uniform interface is partially addressed by the combination of URIs and HTTP verbs and using them in line with the standards and conventions.

**Some tips are:**

* In deciding what resources are within your system, name them as nouns as opposed to verbs or actions. In other words, a RESTful URI should refer to a resource that is a thing instead of referring to an action.
* Resource in a service suite will have at least one URI that URIs should follow a predictable, hierarchical structure to enhance understandability and therefore usability.
* Predictable in the sense that they're consistent, hierarchical in the sense that data has structure relationships. This is not a REST rule or constraint, but it enhances the API.

RESTful APIs are written for consumers. The name and structure of URIs should convey meaning to those consumers.
Let's say we're describing an order system with customers, orders, line items, products, etc. Consider the URIs involved in describing the resources in this service suite.

### Resource Naming Anti-Patterns
It's informative to see some anti-patterns so that we can make sure what not to do.
Below are some examples of poor RESTful resource URIs.

1. Services use a single URI to specify the service interface, using query-string parameters to specify the requested operation and/or HTTP verb.
For example,
 To update customer with ID 12345, the request for a JSON body might be:
 * GET http://api.example.com/services?op=update_customer&id=12345&format=json
 * Though the 'services' URL node is a noun, this URL is not self- descriptive
 * It uses GET as the HTTP verb even though we're performing an update
2. Here's another example following the same operation of updating a customer:
 * GET http://api.example.com/update_customer/12345
 * GET http://api.example.com/customers/12345/update
