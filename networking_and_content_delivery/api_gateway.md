# API Gateway

A Fully-managed service for creating and maintaining REST and WebSocket APIs. API Gateway is serverless. API Gateway can also be configured as a SOAP Webservice passthrough (this is some ancient tech right here).

API endpoints can be cached to accommodate for frequent similar requests.

# Creating a REST API
## Set up REST API Methods
An API method embodies a *method request* and a *method response*. For input, you choose the method request parameters and applicable payload. For output, you determine the method response status code, headers, and applicable body to map the backend response data into.

## Gateway Responses
| Gateway response type	| Default status code | Description |
| ------------- | ------------- | ------------- |
| INTEGRATION_FAILURE  | **504**  | Integration failed. |
| INTEGRATION_TIMEOUT  | **504**  | Integration took too long to respond. |

## Importing APIs
The **API Gateway Import** feature can import an API from an external Swagger v2.0 definiton file. This can be done via the AWS Console or through a HTTP request; a POST request with the Swagger definition in the payload will create a new API, and a PUT request will update an existing API.

# Deploying a REST API
## Stages
A *stage* is a named reference for a version of a deployed API.

Stages can have *stage variables*, which are name-value pairs that you can define as configuration attributes associated with a deployment stage of a REST API. They act like environment variables and can be used in your API setup and mapping templates. 

## API Caching
When *API Caching* is enabled, API Gateway caches endpoint responses for a specified TTL period (default 3600s). When a request is made to the API, API Gateway responds by looking up the endpoint response from the cache instead of making a request to the endpoint.

A client can invalidate an existing cache entry and reload it from the integration endpoint to receive the latest data, provided the client is authorized in IAM to do so. The client's request must contain the `Cache-Control: max-age=0` header.

## Throttling
Default limits for API Gateway are 10,000 requests per second or 5000 concurrent requests across *all* APIs on an AWS account. Exceeding this limit results in a **429 Too Many Requests** error.


