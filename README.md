# Overview
This describes the resources that make up the iProva API v1. This page is structered in a way that the most important information is presented on top. The actual routes are all documented in [Swagger][swagger_v1]. The most recent version of the swagger.json can be found [here][swaggerjson].

We are very interested in your opinion about the iProva REST API. Please [mail][api_email] us comments, problems or questions so we can keep improving our API, documenation and the technical implementation.

## Versioning
By default, all requests receive the latest version of the iProva API. Currently this is v1. We encourage to explicitly request this version via one of the following ways:

- Via the accept header: `Accept: application/vnd.iprova.api+json+api-version=1`.
- Via a custom header: `x-Api-version: 1`.
- Via the query string: `api/card_file/cards?api-version=1`.

The major versions might not be completely backwards compatible with older major versions. Minor versions denote only extensions in the API. The changelog can be found at [this location][change_log], it contains information about the changes and also will describe candidates which are marked to become deprecated in the following major version.

The API version can be retrieved by calling: `GET api/versions/api`.

## Schema
All API access is possible over the same protocols as the iProva.
```javascript
var iprova = "https://iprova.yourcompany.nl";
var api = "https://iprova.yourcompany.nl/api";
```

Blank fields are included as `null` instead of being omitted.
```javascript
var card = 
{
	"card_id" : 22,
	"image" : null
};
```

All resources and attributes of the resources are snake cased.
```javascript
var data_type = 
{
	"data_type_id" : 512,
	"singular_name" : "Car",
	"plural_name" : "Cars",
	"active" : true
};
```

Dates are returned in format `yyyyMMdd` and date with times are returned in format `yyyyMMddHHmmss`.
```javascript
var example = 
{
	"inserted_date" : 19830409,
	"start_datetime" : 200604240830
};
```

### Summary representations
When you fetch a list of resources, the response includes a subset of the attributes for that resource. This is the "summary" representation of the resource. Some attributes are computationally expensive for the API to provide. For performance reasons, the summary representation excludes those attributes. To obtain those attributes, fetch the "detailed" representation or, when supported, pass `include_<attribute>=true` via the query string.

**Example**: When you get a list of cards the image attribute is not filled because this would send the image as a Base64 string. You can include the image by including a query string parameter `GET api/card_file/data_types/1/cards?include_image=true`.

### Detailed representations
When you fetch an individual resource, the response typically includes all attributes for that resource. This is the "detailed" representation of the resource. 

**Example**: When you get an individual repository, you get the detailed representation of the repository. Here, we fetch the a card `GET api/card_file/card/1`.

## Parameters
Many API methods take optional parameters. For GET requests, any parameters not specified as a segment in the path can be passed as an HTTP query string parameter. For example: `GET api/card_file/card/1?include_image=true`.

For POST, PUT, PATCH, and DELETE requests the model parameter should be put in the body. They should be encoded as JSON with a Content-Type of 'application/json'. For example: `POST api/card_file/card/1/image` with body `'{"name":"Hammer", "base64":""}'`.

## HTTP Verbs
The following verbs are used in the API. See the [Verbs][verbs] page for further information.

| Verb | Explanation |
|--|--|
| **GET** | Used for retrieving resources. |
| **POST** | Used for creating resources. |
| **PUT** | Used for replacing resources.|
| **PATCH** | Used for updating resources with partial data. |
| **DELETE** | Used for deleting resources. |

## HTTP Status Codes
The following HTTP status codes can be returned by the services. Check the documentation to know which status code will be returned by which route.

|Code|Name|Explanation|
|--|--|--|
|**200**|OK|Always returned when route did not create resources and a response payload is returned.|
|**201**|Created|Returned when one or more resources are created, a response payload should return (links to) the created resources.|
|**202**|Accepted|Asynchronous route is accepted. Used for fire and forget routes.|
|**204**|No Content|Returned when route did not create resources and no response payload returned.|
|**400**|Bad Request|Returned when any of the input is wrong or a combination of input would cause an illegal operation.|
|**401**|Unauthorized|Returned when anything with the credentials is wrong. It is always possible to receive this status code.|
|**403**|Forbidden|Returned when the authenticated user is forbidden to use a certain aspect of a route.|
|**404**|Not Found|Returned when a resource could not be found.|
|**500**|Internal Server Error|Returned whenever this occurs something is wrong with the API. It is always possible to receive this status code.|

## Client Errors
When a 4xx or 500 HTTP status code is returned the reason phrase is set to be as specific as possible without exposing too much information. See the [Client errors][client_errors] page for detailed information per client error.

## Authentication
There are three ways to authenticate yourself iProva API v1. When the authentication fails a 401 Unauthorized HTTP status code wil be returned.

### API Keys
To access the API, you need an API-Key. In iProva we have two different kinds of API keys. One that allows you to impersonate any given iProva user, and one that simply allows you to access the API. The first one is used for Token authentication. The second one is used for credentials authentication.

### Via Token (preferred authentication method)
The token can be sent via the Authorization header with the string "token" followed by the token id. `Authorization: token e8f66f95-7ab2-404e-b557-879788b900de`. 
For more information about token authentication see [Tokens][Tokens]

### Via Credentials
If the username and password of a user are known these credentials can be directly used to authenticate the user via the Authorization header. The header should contain the string "credentials" followed by the string "u:" and the username, a whitespace, the string "pwd:" and the password. `Authorization: credentials u:j.t.kirk pwd:P@$$w0rd`. 

In this situation, passing an API key is still required. The API key can be passed via the "api_key" querystring parameter, or via an "api_key" http header.

Of course the consumer should keep in mind that this would require the password to be sent via a http header, so only use this in combination with HTTPs.

### Via iProva Cookie
When the user is already logged in in iProva, iProva has set an authentication cookie in the browser. When accessing the API when this cookie is set the API will automatically authenticate you using this cookie.

## Pagination
Some api paths have been implemented using paginated results. This means that when getting the results, you only get a subset of the result, representing a single page of results. You can influence the data being returned by using the "limit" and "offset" querystring parameters. 

**Example**: `GET api/card_file/data_types/1/cards?limit=50&offset=10` 

This call will return (at most) 50 cards, starting at card number 11.

The following metadata will be included in the result:

- The total number of results of the request (that would be returned if the result was not paginated)
- The used limit parameter
- The used offset parameter, determining how many results in the entire resultset to skip in the returned result
- The amount of results returned

By default the paging metadata will be returned as custom http response headers:

```bash
HTTP/1.1 200 OK
Date: Thu, 02 Mar 2017 17:27:06 GMT
Status: 200 OK
X-Pagination-Limit: 50
X-Pagination-Offset: 10
X-Pagination-Returned: 50
X-Pagination-Total: 1048
```

However, because some proxy servers don't allow unknown headers and remove them from the response, and some client might not be able to access the response headers it is possible to get this metadata in the actual result of the call. This can be done by passing the `envelope` query string parameter and setting it to true. 

**Example**: `GET api/card_file/data_types/1/cards?limit=50&offset=10&envelope=true`

The result of this call will always be a generic `paging_envelope`. This envelope contains two properties: "data" and "pagination". "data" contains the actual result of the request, and "pagination" contains the metadata about the pagination that would normally be present in the response headers:

```javascript
{
  "data" : [the actual result array]
  "pagination": {
    "limit": 3,
    "offset": 0,
    "returned": 3,
    "total": 6
  }
}
```

[//]: # (These are reference links used in the body of this note and get stripped out when the markdown processor does its job. There is no need to format nicely because it shouldn't be seen.)
[change_log]: <changelog.md>
[swagger_v1]:<../swagger>
[swaggerjson]:../swagger/docs/v1
[tokens]: <Tokens.md>
[api_email]: <mailto:s.v.loon@infoland.nl>
[verbs]: <Verbs.md>
[client_errors]:<ClientErrors.md>
[examples]:<https://github.com/Infoland/iProva-REST-API-examples>