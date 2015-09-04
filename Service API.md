# Hone cloud service API

This is the API implemented by Hone cloud service, with API endpoints running at `https://hone.tools/v1/`.

The API consists of two parts. **Project API** works in the context of one project. The desktop tool puts Hone documents to the API, and clients can retrieve current values and assets.

The goal of the project API is to let users test the values encoded in Hone documents on client apps and devices. It is explicitly not designed to be a version control system: for this purpose, real version control systems (Git etc) should be used to manage the documents. Most PUT requests unconditionally overwrite the current state of a resource.

The other part of the API, **management API**, deals with creating new projects and seeing the associated metadata (authorized users, logs etc). The desktop tool uses this API to manage the projects.



## Core concepts

### Authentication

Authentication is implemented with OAuth2 Bearer tokens as specified in [RFC6750](http://tools.ietf.org/html/rfc6750). The token can be passed either as Authorization header, or as access_token parameter to the request.

There are two levels of access which correspond to two kinds of Bearer tokens: user tokens and document tokens.

**User tokens** identify and authenticate a particular end user of Hone who has a user account on hone.tools. You can obtain your user token on [hone.tools/you](https://hone.tools/you/). The user token provides full access to both project API and management API.

**Project tokens** are associated with one Hone project, and provide read-only access to requests regarding one project using the project API. Project tokens should be used as authenticators in the Hone device libraries. You can obtain the project token from the page of the individual project.

Authentication examples:

#### Request

    GET /v1/projects/53551a31e9cb4e000027f3f8/manifest?access_token=4T%2BwCx3LXFNdCEsPAgRFZtleO30tX0xeR4O0BlrNZx4%3D

    GET /v1/projects/53551a31e9cb4e000027f3f8/manifest
    Authorization: Bearer 4T+wCx3LXFNdCEsPAgRFZtleO30tX0xeR4O0BlrNZx4=

#### Response

 * `400 Bad Request` The request was malformed, e.g the token was specified both as the header and parameter.
 * `401 Unauthorized` The request was valid, but this token does not authorize a user to access this project.
 * If the authorization is valid, see individual API endpoints for possible responses.



### Errors

If a response to a given request is an error (anything other than HTTP 200 or 300 series), this is reflected by the HTTP status code (400 series: client errors, 500 series: server errors), as well as a JSON dictionary in the response body, containing info about it in the error dictionary. "error.name" is a machine-readable error name and can be expected to stay consistent over localizations and service versions, while error.message is a human-readable name that may change frequently, be localized etc.

    {
      "error":
        { "name": "DocumentNotFoundError",
         "message": "This document does not exist." }
    }



### Caching and ETags

The current state of many resources in the Hone system is indicated by a base64-encoded hash of its contents, such as seen in the manifest file. You should use this hash as an ETag of your requests for this resource, to cut down unnecessary traffic.


## Project API

### Manifest

The manifest is a key resource of each Hone document, containing metadata as well as a listing of other resources and their checksums. A complete representation of a Hone document can be obtained by first retrieving the manifest, and then the resources that it refers to.



#### Get manifest

    GET /v1/projects/53551a31e9cb4e000027f3f8/manifest

Access level: user token or project token

Response:

 * `404 Not Found.` This document was not found.
 * `304 Not Modified.` An ETag was included in the request, and the manifest on the server has the same ETag, i.e the client already has the current version.
 * `204 No Content.` No manifest has yet been uploaded for this project.
 * `200 OK.` Everything is fine with the manifest. The document body contains the manifest content in YAML format, verbatim as it was previously uploaded. The content type is application/x-yaml.



#### Put manifest

    PUT /v1/projects/53551a31e9cb4e000027f3f8/manifest
    Content-type: application/x-yaml

Access level: user token

Response:

 * `200 OK.` All good, current version of the manifest was put. Additionally, the body contains the resources that the server is missing, similarly as missing_resources response. Response content type is application/json. The header also contains ETag for the manifest.
 * `400 Bad Request.` The document was possibly malformed. The JSON error body contains more information. Response content type is application/json.
 * `204 No Content`. Manifest was successfully received, but all resources are already correctly represented on the server.
 * `304 Not Modified`. The version of the manifest on the server is the same one that is already there, so the manifest wasn’t stored. (Note that this doesn’t tell you anything about missing resources.)



#### Get missing resources for manifest

It is possible that a manifest file has been uploaded to the server, but some of the referred resources are missing. The client can ask the server any time what resources it is missing.

    GET /v1/projects/53551a31e9cb4e000027f3f8/manifest/missing_resources

Access level: user token or project token

Response:

 * `200 OK.` The list of missing resources is returned as JSON object, with the key "missing_resources" whose value is an array of the missing resource dictionaries. The result is JSON because it’s not versioned/checksummed, it is just an ephemeral item based on the current state of the system.

    ```
    { "missing_resources": [
      { "name": "values.yaml", "theme": "default", "checksum": "VWUpRQ0GDLcorRT+a0wJsB1o0OU2M8CQeUSjmLAwvgg=" },
      { "name": "values.yaml", "theme": "anotherTheme", "checksum": "4T+wCx3LXFNdCEsPAgRFZtleO30tX0xeR4O0BlrNZx4=" }
    ]}
    ```

 * `204 No Content`. All resources are correctly represented on the server.

If a reference to `aliases.yaml` is present in the manifest, but the aliases file has not yet been uploaded, the `missingResources` response will include that as well.

```
{ "missing_resources": [
  { "name": "aliases.yaml", "theme": "", "checksum": "xEZBszGCykswqQ7jTbvbyaOSVXsTkPIWjE1bQ1vPOMs=" },
  { "name": "values.yaml", "theme": "default", "checksum": "VWUpRQ0GDLcorRT+a0wJsB1o0OU2M8CQeUSjmLAwvgg=" },
  { "name": "values.yaml", "theme": "anotherTheme", "checksum": "4T+wCx3LXFNdCEsPAgRFZtleO30tX0xeR4O0BlrNZx4=" }
]}
```


### Aliases

The optional aliases.yaml file defines object and value name aliases that, unlike the original object and value names, can be modified by the end user at any time using the Hone editor tool. See the [document format](Document.md) for more details about aliases.

These simple endpoints let you get or put the aliases content.

#### Get aliases

    GET /v1/projects/53551a31e9cb4e000027f3f8/aliases

Access level: user token or project token

Response:

* `404 Not Found.` This document was not found.
* `304 Not Modified.` An ETag was included in the request, and the manifest on the server has the same ETag, i.e the client already has the current version.
* `200 OK.` Everything is fine with the manifest. The document body contains the aliases content in YAML format, verbatim as it was previously uploaded. The content type is application/x-yaml.



#### Put aliases

    PUT /v1/projects/53551a31e9cb4e000027f3f8/manifest
    Content-type: application/x-yaml

Access level: user token

Response:

* `200 OK.` All good, current version of the aliases was put. The header also contains ETag for the aliases.
* `400 Bad Request.` The document was possibly malformed. The JSON error body contains more information. Response content type is application/json.
* `304 Not Modified`. The version of the aliases on the server is the same one that is already there, so the aliases weren’t stored.



### Individual resources

These are the values.yaml files of each theme contained in the document, and in the future, also other assets (images, sounds, fonts etc) contained in each theme. The PUT and GET commands for them are pretty straightforward.


#### Put resource

Note how the theme and resource file name are simply part of the URL.

    PUT /v1/projects/53551a31e9cb4e000027f3f8/resources/default/values.yaml

Access level: user token

Request body is the content of the resource.

Response:

  * `200 OK.` The resource was successfully put. The `ETag` header also contains the resource ETag.
  * `404 Not Found.` The project was not found.
  * `409 Conflict.` Either there is no manifest for this project, or the manifest does not contain a reference to this resource. Hone requires that all uploaded resources are referred to in the manifest. This probably means that you are trying to upload a resource before uploading an up-to-date manifest. See the `error` object in the body for details.

#### Get resource

Whenever possible, include an If-None-Match header in the request, to cut down unnecessary traffic: if you already have the same version of the resource as is on the server, `304 Not Modified` is returned.

    GET /v1/projects/53551a31e9cb4e000027f3f8/resources/default/values.yaml
    If-None-Match: VWUpRQ0GDLcorRT+a0wJsB1o0OU2M8CQeUSjmLAwvgg=

Access level: user token or project token

Response

 * `200 OK.` The resource was retrieved successfully, the body contains the resource content.
 * `304 Not Modified.` The resource representation on the server is the same as specified by the ETag, i.e the server and client both have the same resource and it does not need to be transmitted.
 * `404 Not Found.` The resource or project was not found.



## Project management API

You use the project management API to retrieve the list of projects for a given user, create new ones, and retrieve and change metadata regarding a project, such as the project name or list of authorized users.

Deleting a project is currently not supported in the management API. You must use the Hone web UI for this.

### Get projects

    GET /v1/projects

Access level: user token

Response

`200 OK.` List of projects for the user. Also contains the authorized users for each project, and the latest log entry.

      [
        {
          "name": "My Awesome App",
          "id": "53551a31e9cb4e000027f3f8",
          "vcs": "github",
          "platform": "ios",
          "vcsUrl": "https://github.com/someone/exampleProject",
          "users": [
            {
              "id": "53551a31e9cb4e0000271234",
              "name": "Bob Yerunkel",
              "email": "bob@example.com"
            },
            {
              "id": "53551a31e9cb4e0000274567",
              "name": "Ally Gator",
              "email": "ally@example.com"
            }
          ],
          "latestLog": {
            "project": {
              "\_id": "53551a31e9cb4e000027f3f8",
              "name": "My Awesome App"
            },
            "user": {
              "\_id": "53551a31e9cb4e0000271234",
              "email": "bob@example.com",
              "name": "Bob Yerunkel"
            },
            "event": "userPutProjectResource",
            "resource": {
              "theme": "default",
              "name": "values.yaml"
            },
            "time": "2014-09-02T21:20:16.210Z",
            "affectedUsers": []
          }
        },
        {
          "name": "Another Awesome App",
          "id": "53551a31e9cb4e000027bee1",
          "users": [
            {
              "id": "53551a31e9cb4e0000271234",
              "name": "Bob Yerunkel",
              "email": "bob@example.com"
            },
            {
              "id": "53551a31e9cb4e0000274567",
              "name": "Ally Gator",
              "email": "ally@example.com"
            }
          ],  
          "latestLog": {
            "project": {
              "id": "53551a31e9cb4e000027bee1",
              "name": "Another Awesome App"
            },
            "user": {
              "id": "53551a31e9cb4e0000274567",
              "email": "ally@example.com",
              "name": "Ally Gator"
            },
            "event": "userPutProjectResource",
            "resource": {
              "theme": "default",
              "name": "values.yaml"
            },
            "time": "2014-09-02T21:20:16.210Z",
            "affectedUsers": []
          }
        }
      ]

### Metadata for one project

    GET /v1/projects/53551a31e9cb4e000027f3f8

Access level: user token

Response

`200 OK.` Metadata for this project in JSON format (including creator, authorized users, and latest log entry).

      {
        "name": "My Awesome App",
        "id": "53551a31e9cb4e000027f3f8",
        "vcs": "github",
        "vcsUrl": "https://github.com/someone/exampleProject"
        "users": [
          {
            "id": "53551a31e9cb4e0000271234",
            "name": "Bob Yerunkel",
            "email": "bob@example.com"
          },
          {
            "id": "53551a31e9cb4e0000274567",
            "name": "Ally Gator",
            "email": "ally@example.com"
          }
        ],
        "latestLog": {
          "project": {
            "id": "53551a31e9cb4e000027f3f8",
            "name": "My Awesome Project"
          },
          "user": {
            "id": "53551a31e9cb4e0000271234",
            "email": "bob@example.com",
            "name": "Bob Yerunkel"
          },
          "event": "userPutProjectResource",
          "resource": {
            "theme": "default",
            "name": "values.yaml"
          },
          "time": "2014-09-02T21:20:16.210Z",
          "affectedUsers": []
        }
      }


### Create a project

    POST /v1/projects

Access level: user token

Request body should contain the project name encoded in JSON.

    {
      "name": "Great App"
    }

Optionally, there may be a `vcsUrl` specified, pointing to a valid Github repository URL

    {
      "name": "Great App",
      "vcsUrl": "https://github.com/someone/exampleProject"
    }

There can also be a `platform` specified, which can be one of `ios`, `osx` or `android`. There can also be no platform, indicated by an empty or missing platform value.

    {
      "name": "Mac App",
      "platform": "osx"
    }



Response

`200 OK.` The project was created. The response contains the project name, ID and other metadata.

    {
      "name": "Great App",
      "id": 53551a31e9cb4e000027abcd,
      "platform": ""
    }

`400 Bad Request`. The project could not be created for some reason, e.g no name was supplied, or vcsUrl was supplied but was invalid, or platform was supplied but was invalid. The error response contains more details.


### Delete a project

    DELETE /v1/projects/53551a31e9cb4e000027f3f8

Access level: user token

Response

* `200 OK.` The project was deleted.
