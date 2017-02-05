---
title: Data API

language_tabs:
  - shell

toc_footers:
  - <a href='https://sharemy3d.com/#plans'>Sign Up for a Developer Key</a>

search: true
---

# Introduction

ShareMy3D provides the Data API as a way to programmatically interact with your ShareMy3D account. This API is the same API we use internally to power our dashboard, so anything we can do, you can do too! The Data API is organized around [REST](https://en.wikipedia.org/wiki/Representational_state_transfer). We support [cross-origin resource sharing](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing), allowing you to interact securely with our API from a client-side web application (though you should never expose your secret API token in any public website's client-side code). [JSON](http://www.json.org/) is returned by all API responses, including errors.

# Authentication

> To authorize, use this code:

```shell
# With shell, you can just pass the correct header with each request
curl "API_ENDPOINT_HERE"
  -H "Authorization: 00xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

> Make sure to replace `00xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx` with your API token.

The API uses API tokens to allow access to the API. You can retrieve your API token in your [Account Settings](https://sharemy3d.com/dashboard/account).
Do not share your secret API keys in publicly accessible areas such GitHub, client-side code, and so forth.

The Data API expects the API token to be included in all API requests to the server in a header that looks like the following:

`Authorization: 00xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`

<aside class="notice">
You must replace <code>00xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx</code> with your personal API token.
</aside>

All API requests must be made over [HTTPS](https://en.wikipedia.org/wiki/HTTPS). Calls made over plain HTTP will fail. API requests without authentication will also fail.

# Errors

The Data API uses conventional HTTP response codes to indicate the success or failure of an API request. In general, codes in the `2xx` range indicate success, codes in the `4xx` range indicate an error that failed given the information provided (e.g., a required parameter was omitted, a paramater was invalid, etc.), and codes in the `5xx` range indicate an error with ShareMy3D's servers (these are rare).

**HTTP status code summary**

HTTP Status Code | Meaning
---------- | -------
200, 201 | OK -- Everything worked as expected.
400 | Bad Request -- The request was unacceptable, often due to missing a required parameter.
401 | Unauthorized -- Your API token is invalid or access denied.
404 | Not Found -- The endpoint / resource was not found.
429 | Too Many Requests -- Too many requests hit the API too quickly. We recommend an exponential backoff of your requests.
500, 503 | Server errors -- Something went wrong on ShareMy3D's end. (These are rare.)

# Models

## Upload new model

```shell
curl "https://api.sharemy3d.com/v1/models/upload" \
  -H "Authorization: 003b3c2c582348b084394435b1c4e39a" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{
        "size": 123,
        "filename": "abc.ply",
        "name": "Test model"
      }'
```

> The above command returns JSON structured like this:

```json
{  
  "class": "UploadModelResponse",
  "modelId": "[MODEL_ID]",
  "uploadUrl": "[UPLOAD_URL]"
}
```

> Then you perform the actual upload to the **UPLOAD_URL** like this:

```shell
curl \
  -X PUT \
  -T - "[UPLOAD_URL]" \
  < PATH_TO_LOCAL_FILE
```

This endpoint creates a new model.

### HTTP Request

`POST https://api.sharemy3d.com/v1/models/upload`

### Arguments

Parameter | Default | Required | Description
--------- | ------- | ------- | -----------
size | | yes | Size in bytes (integer) of the model file to be uploaded.
filename | | yes | Filename (string) of the original file (ex: "**Book.ply**"). This name will be used when downloading original file, and the extension is used as format identifier. **Length range: 0-100**.
name | | yes | Name (string) of the model. **Length range: 3-50**
description | | no | Description (string) of the model. **Length range: 0-100**
password | | no | If set then this password (string) is required for viewing the model. **Length range: 0-100**
downloadable | false | no | If the model should be downloadable or not.
resumable | false | no | If the model file should be uploaded by chunks (enables resumable upload). See: [Resumable upload](https://cloud.google.com/storage/docs/json_api/v1/how-tos/resumable-upload) (under "Uploading the file").

## Retrieve a model

```shell
curl "https://api.sharemy3d.com/v1/models/9e4d36b157df466da6dd9d32956561e1/info" \
  -H "Authorization: 003b3c2c582348b084394435b1c4e39a"
```

> The above command returns JSON structured like this:

```json
{
  "class": "RetrieveModelResponse",
  "modelId": "9e4d36b157df466da6dd9d32956561e1",
  "name": "Test model",
  "description": "",
  "hasPassword": false,
  "downloadable": false,
  "timeCreated": 1485557110,
  "status": "PROCESSING",
  "size": 123,
  "numViews": 0
}
```

This endpoint retrieves information about a model.

### HTTP Request

`GET https://api.sharemy3d.com/v1/models/:modelId/info`

### Attributes
Attribute | Type | Description
--------- | ---- | -----------
modelId | string | Unique model identifier.
name | string | Model's name.
description | string | Model's description.
hasPassword | bool | If the model is password protected.
hasPassword | bool | If the model is downloadable.
timeCreated | timestamp | When the model was created (Unix timestamp).
status | string | One of <ul><li>UPLOADING</li><li>PROCESSING</li><li>ACTIVE</li><li>FAILED</li></ul>
size | number | File size (in bytes) of the original file.
numViews | number | Number of model views.

## List all models

```shell
curl "https://api.sharemy3d.com/v1/models?limit=3&offset=6&sort=-name" \
  -H "Authorization: 003b3c2c582348b084394435b1c4e39a"
```

> The above command returns JSON structured like this:

```json
{  
  "class": "GetModelListResponse",
  "models": [  
    {  
      "modelId": "9e4d36b157df466da6dd9d32956561e1",
      "name": "Test model",
      "description": "",
      "hasPassword": false,
      "downloadable": false,
      "timeCreated": 1485557110,
      "status": "PROCESSING",
      "size": 123,
      "numViews": 0
    },
    {...},
    {...}
  ],
  "meta": {  
    "totalCount": 35,
    "first": "https://api.sharemy3d.com/v1/models?limit=3&offset=0&sort=-name",
    "prev": "https://api.sharemy3d.com/v1/models?limit=3&offset=3&sort=-name",
    "next": "https://api.sharemy3d.com/v1/models?limit=3&offset=9&sort=-name",
    "last": "https://api.sharemy3d.com/v1/models?limit=3&offset=33&sort=-name"
  }
}
```

This endpoint retrieves a list of all your models (or partially).

### HTTP Request

`GET https://api.sharemy3d.com/v1/models`

### Query parameters
Parameter | Default | Description
--------- | ---- | -----------
limit | 20 | Maximum number of returned items. Limit can range between 1 and 100 items.
offset | 0 | The offset of the first row to return.
status | | If set then the request will filter models by status. A comma-seperated list of accepted model statuses. Valid values are <ul><li>UPLOADING</li><li>PROCESSING</li><li>ACTIVE</li><li>FAILED</li></ul> **Example: `"ACTIVE,FAILED"`**
sort | time | A comma-seperated list of sorting orders. Valid values are: <ul><li>time (timeCreated, ascending)</li><li>-time (timeCreated, descending)</li><li>name (ascending)</li><li>-name (descending)</li><li>views (numViews, ascending)</li><li>-views (numViews, descending)</li></ul> **Example: `"-name,views"`**

## Delete a model

```shell
curl "https://api.sharemy3d.com/v1/models/9e4d36b157df466da6dd9d32956561e1" \
  -H "Authorization: 003b3c2c582348b084394435b1c4e39a" \
  -X DELETE
```

> The above command returns JSON structured like this:

```json
{
  "class": "DeleteModelResponse"
}
```

This endpoint deletes a model.

### HTTP Request

`DELETE https://api.sharemy3d.com/v1/models/:modelId`

<aside class="warning">After a successful request you will no longer be able to access or edit the model.</aside>

## Download original file

```shell
curl "https://api.sharemy3d.com/v1/models/9e4d36b157df466da6dd9d32956561e1/original" \
  -H "Authorization: 003b3c2c582348b084394435b1c4e39a"
```

> The above command returns JSON structured like this:

```json
{  
  "class": "DownloadOriginalResponse",
  "downloadUrl": "DOWNLOAD_URL"
}
```

This endpoint generates a time limited URL to download the original file.

### HTTP Request

`GET https://api.sharemy3d.com/v1/models/:modelId/original`

# Backgrounds

## Upload new background

```shell
curl "https://api.sharemy3d.com/v1/backgrounds/upload" \
  -H "Authorization: 003b3c2c582348b084394435b1c4e39a" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{
        "size": 123,
        "filename": "landscape.jpg",
        "name": "Landscape",
        "type": "IMAGE"
      }'
```

> The above command returns JSON structured like this:

```json
{  
  "class":"UploadBackgroundResponse",
  "backgroundId":"[BACKGROUND_ID]",
  "uploadUrl":"[UPLOAD_URL]"
}
```

> Then you perform the actual upload to the **UPLOAD_URL** like this:

```shell
curl \
  -X PUT \
  -T - "[UPLOAD_URL]" \
  < PATH_TO_LOCAL_FILE
```

This endpoint creates a new background.

### HTTP Request

`POST https://api.sharemy3d.com/v1/backgrounds/upload`

### Arguments

Parameter | Default | Required | Description
--------- | ------- | ------- | -----------
type | | yes | Background type. Allowed values: `"ÌMAGE"`, `"PANORAMA"`.
size | | yes | Size in bytes (integer) of the image file to be uploaded.
filename | | yes | Filename (string) of the original file (ex: "**Landscape.jpg**"). The file extension is used as format identifier. **Length range: 0-100**.
name | | yes | Name (string) of the background. **Length range: 3-50**
description | | no | Description (string) of the model. **Length range: 0-100**
resumable | false | no | If the model file should be uploaded by chunks (enables resumable upload). See: [Resumable upload](https://cloud.google.com/storage/docs/json_api/v1/how-tos/resumable-upload) (under "Uploading the file").

## Retrieve a background

```shell
curl "https://api.sharemy3d.com/v1/backgrounds/9e4d36b157df466da6dd9d32956561e1/info" \
  -H "Authorization: 003b3c2c582348b084394435b1c4e39a"
```

> The above command returns JSON structured like this:

```json
{  
  "class": "RetrieveModelResponse",
  "backgroundId": "d652d493808049e283315d133c328d42",
  "type": "IMAGE",
  "name": "Landscape",
  "status": "ACTIVE",
  "timeCreated": 1485773609
}
```

This endpoint retrieves information about a background.

### HTTP Request

`GET https://api.sharemy3d.com/v1/models/:modelId/info`

### Attributes
Attribute | Type | Description
--------- | ---- | -----------
backgroundId | string | Unique background identifier.
type | string | Type of background. Possible values: `"IMAGE"`, `"PANORAMA"`.
name | string | Background's name.
status | string | One of <ul><li>UPLOADING</li><li>PROCESSING</li><li>ACTIVE</li><li>FAILED</li></ul>
timeCreated | timestamp | When the background was created (Unix timestamp).

## List all backgrounds

```shell
curl "https://api.sharemy3d.com/v1/backgrounds?limit=3&offset=6&sort=-time" \
  -H "Authorization: 003b3c2c582348b084394435b1c4e39a"
```

> The above command returns JSON structured like this:

```json
{
  "class": "GetBackgroundListResponse",
  "backgrounds": [
    {
      "backgroundId": "d652d493808049e283315d133c328d42",
      "type": "IMAGE",
      "name": "Landscape",
      "status": "ACTIVE",
      "description": null,
      "timeCreated": 1485773609
    },
    {...}
  ],
  "meta": {
    "totalCount": 8,
    "first": "https://api.sharemy3d.com/v1/backgrounds?limit=3&offset=0&sort=-time",
    "prev": "https://api.sharemy3d.com/v1/backgrounds?limit=3&offset=3&sort=-time",
    "next": null,
    "last": "https://api.sharemy3d.com/v1/backgrounds?limit=3&offset=6&sort=-time"
  }
}
```

This endpoint retrieves a list of all your background (or partially).

### HTTP Request

`GET https://api.sharemy3d.com/v1/backgrounds`

### Query parameters
Parameter | Default | Description
--------- | ---- | -----------
limit | 20 | Maximum number of returned items. Limit can range between 1 and 100 items.
offset | 0 | The offset of the first row to return.
status | | If set then the request will filter backgrounds by status. A comma-seperated list of accepted background statuses. Valid values are <ul><li>UPLOADING</li><li>PROCESSING</li><li>ACTIVE</li><li>FAILED</li></ul> **Example: `"ACTIVE,FAILED"`**
sort | time | A comma-seperated list of sorting orders. Valid values are: <ul><li>time (timeCreated, ascending)</li><li>-time (timeCreated, descending)</li><li>name (ascending)</li><li>-name (descending)</li></ul> **Example: `"-time,-name"`**

## Delete a background

```shell
curl "https://api.sharemy3d.com/v1/backgrounds/d652d493808049e283315d133c328d42" \
  -H "Authorization: 003b3c2c582348b084394435b1c4e39a" \
  -X DELETE
```

> The above command returns JSON structured like this:

```json
{
  "class": "DeleteBackgroundResponse"
}
```

This endpoint deletes a background.

### HTTP Request

`DELETE https://api.sharemy3d.com/v1/background/:backgroundId`

<aside class="warning">After a successful request you will no longer be able to access or edit the background.</aside>
<aside class="warning">Models that are using this background will lose its background.</aside>
