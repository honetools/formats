# Sway cloud service API

This is the API implemented by Sway cloud service, with API endpoints running at `https://swayapp.net/v1/`. The desktop tool puts Sway documents to the API, and clients can retrieve current values and assets.

The goal of the cloud service is to let users test the values encoded in Sway documents on client apps and devices. It is explicitly not designed to be a version control system: for this purpose, real version control systems (Git etc) should be used to manage the documents. Most PUT requests unconditionally overwrite the current state of a resource.



## Core concepts

### Authentication

Authentication is implemented with OAuth2 Bearer tokens as specified in [RFC6750](http://tools.ietf.org/html/rfc6750). The token can be passed either as Authorization header, or as access_token parameter to the request.

You can obtain the tokens from the [Sway web app](https://swayapp.net/apps). All tokens are unique for each user-app combination, i.e they are valid only for this particular Sway user accessing this particular app.

Authentication examples:

#### Request

    GET /v1/apps/53551a31e9cb4e000027f3f8/manifest?access_token=4T%2BwCx3LXFNdCEsPAgRFZtleO30tX0xeR4O0BlrNZx4%3D

    GET /v1/apps/53551a31e9cb4e000027f3f8/manifest
    Authorization: Bearer 4T+wCx3LXFNdCEsPAgRFZtleO30tX0xeR4O0BlrNZx4=

#### Response

 * `400 Bad Request` The request was malformed, e.g the token was specified both as the header and parameter.
 * `401 Unauthorized` The request was valid, but this token does not authorize a user to access this app.
 * If the authorization is valid, see individual API endpoints for possible responses.



### Errors

If a response to a given request is an error (anything other than HTTP 200 or 300 series), this is reflected by the HTTP status code (400 series: client errors, 500 series: server errors), as well as a JSON dictionary in the response body, containing info about it in the error dictionary. "error.name" is a machine-readable error name, while error.message is a human-readable name that may change frequently, be localized etc.

    {
      "error":
        { "name": "DocumentNotFoundError",
         "message": "This document does not exist."
    }



### Caching and ETags

The current state of many resources in the Sway system is indicated by a base64-encoded hash of its contents, such as seen in the manifest file. You should use this hash as an ETag of your requests for this resource, to cut down unnecessary traffic.


## Manifest

The manifest is a key resource of each Sway document, containing metadata as well as a listing of other resources and their checksums. A complete representation of a Sway document can be obtained by first retrieving the manifest, and then the resources that it refers to.



### Get manifest

    GET /v1/apps/53551a31e9cb4e000027f3f8/manifest

#### Response

 * `404 Not Found.` This document was not found.
 * `304 Not Modified.` An ETag was included in the request, and the manifest on the server has the same ETag, i.e the client already has the current version.
 * `200 OK.` Everything is fine with the manifest. The document body contains the manifest content in YAML format, verbatim as it was previously uploaded. The content type is application/x-yaml.



### Put manifest

    PUT /v1/apps/53551a31e9cb4e000027f3f8/manifest
    Content-type: application/x-yaml

#### Response

 * `200 OK.` All good, current version of the manifest was put. Additionally, the body contains the resources that the server is missing, similarly as missing_resources response. Response content type is application/json. The header also contains ETag for the manifest.
 * `400 Bad Request.` The document was possibly malformed. The JSON error body contains more information. Response content type is application/json.



### Get missing resources for manifest

It is possible that a manifest file has been uploaded to the server, but some of the referred resources are missing. The client can ask the server any time what resources it is missing.

    GET /v1/apps/53551a31e9cb4e000027f3f8/manifest/missing_resources

#### Response

 * `200 OK.` The list of missing resources is returned as JSON object, with the key "missing_resources" whose value is an array of the missing resource dictionaries. The result is JSON because it’s not versioned/checksummed, it is just an ephemeral item based on the current state of the system.

    ```
    { "missing_resources": [
      { "name": "values.yaml", "theme": "default", "checksum": "VWUpRQ0GDLcorRT+a0wJsB1o0OU2M8CQeUSjmLAwvgg=" },
      { "name": "values.yaml", "theme": "anotherTheme", "checksum": "4T+wCx3LXFNdCEsPAgRFZtleO30tX0xeR4O0BlrNZx4=" }
    ]}
    ```

 * `204 No Content`. All resources are correctly represented on the server.



## Individual resources

These are the values.yaml files of each theme contained in the document, and in the future, also other assets (images, sounds, fonts etc) contained in each theme. The PUT and GET commands for them are pretty straightforward.


### Put resource

Note how the theme and resource file name are simply part of the URL.

    PUT /v1/apps/53551a31e9cb4e000027f3f8/resources/default/values.yaml

Request body is the content of the resource.

#### Response

  * `200 OK.` The resource was successfully put. The `ETag` header also contains the resource ETag.
  * `404 Not Found.` The application was not found.

### Get resource

Whenever possible, include an ETag header in the request, to cut down unnecessary traffic: if you already have the same version of the resource as is on the server, `304 Not Modified` is returned.

  GET /v1/apps/53551a31e9cb4e000027f3f8/resources/default/values.yaml
  ETag: VWUpRQ0GDLcorRT+a0wJsB1o0OU2M8CQeUSjmLAwvgg=

#### Response

 * `200 OK.` The resource was retrieved successfully, the body contains the resource content.
 * `304 Not Modified.` The resource representation on the server is the same as specified by the ETag, i.e the server and client both have the same resource and it does not need to be transmitted.
 * `404 Not Found.` The resource or application was not found.
