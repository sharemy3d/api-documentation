---
title: Data API

language_tabs:
  - shell
  - javascript

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the Kittn API! You can use our API to access Kittn API endpoints, which can get information on various cats, kittens, and breeds in our database.

We have language bindings in Shell, Ruby, and Python! You can view code examples in the dark area to the right, and you can switch the programming language of the examples with the tabs in the top right.

This example API documentation page was created with [Slate](https://github.com/tripit/slate). Feel free to edit it and use it as a base for your own API's documentation.

# Authentication

> To authorize, use this code:

```
#bare en test ats
```

```shell
# With shell, you can just pass
# the correct header with each request
curl "api_endpoint_here"
  -H "Authorization: 00xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

```javascript
auth.then((res) => { console.log(res); });
```

> Make sure to replace `00xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx` with your API key.

The API uses API tokens to allow access to the API. You can retrieve your API token in your [Account Settings](https://sharemy3d.com/dashboard/account).

The Data API expects the API token to be included in all API requests to the server in a header that looks like the following:

`Authorization: 00xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`

<aside class="notice">
You must replace <code>00xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx</code> with your personal API token.
</aside>

# Models

## Upload new model

```shell
curl "http://api.localhost:4000/v1/models/upload" \
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

`POST http://api.localhost:4000/v1/models/upload`

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
curl "http://api.localhost:4000/v1/models/9e4d36b157df466da6dd9d32956561e1/info" \
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

`GET http://api.localhost:4000/v1/models/:modelId/info`

### Attributes
Attribute | Type | Description
--------- | ---- | -----------
modelId | string | Unique model identifier.
name | string | Model's name.
description | string | Model's description.
hasPassword | bool | If the model is password protected.
hasPassword | bool | If the model is downloadable.
timeCreated | timestamp | When the model was created.
status | string | One of <ul><li>UPLOADING</li><li>PROCESSING</li><li>ACTIVE</li><li>FAILED</li></ul>
size | number | File size (in bytes) of the original file.
numViews | number | Number of model views.

## List all models

```shell
curl "http://api.localhost:4000/v1/models?limit=3&offset=6&sort=-name" \
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
    "first": "http://localhost:4000/api/v1/models?limit=3&offset=0&sort=-name",
    "prev": "http://localhost:4000/api/v1/models?limit=3&offset=3&sort=-name",
    "next": "http://localhost:4000/api/v1/models?limit=3&offset=9&sort=-name",
    "last": "http://localhost:4000/api/v1/models?limit=3&offset=33&sort=-name"
  }
}
```

This endpoint retrieves a list of all your models (or partially).

### HTTP Request

`GET http://api.localhost:4000/v1/models`

### Query parameters
Parameter | Default | Description
--------- | ---- | -----------
limit | 20 | Maximum number of returned items. Limit can range between 1 and 100 items.
offset | 0 | The offset of the first row to return.
status | | If set then the request will filter models by status. A comma-seperated list of accepted model statuses. Valid values are <ul><li>UPLOADING</li><li>PROCESSING</li><li>ACTIVE</li><li>FAILED</li></ul> **Example: `"ACTIVE,FAILED"`**
sort | time | A comma-seperated list of sorting orders. Valid values are: <ul><li>time (timeCreated, ascending)</li><li>-time (timeCreated, descending)</li><li>name (ascending)</li><li>-name (descending)</li><li>views (numViews, ascending)</li><li>-views (numViews, descending)</li></ul> **Example: `"-name,views"`**

## Delete a model

```shell
curl "http://api.localhost:4000/v1/models/9e4d36b157df466da6dd9d32956561e1" \
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

`DELETE http://api.localhost:4000/v1/models/:modelId`

<aside class="warning">After a successful request you will no longer be able to access or edit the model.</aside>

## Download original file

```shell
curl "http://api.localhost:4000/v1/models/9e4d36b157df466da6dd9d32956561e1/original" \
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

`GET http://api.localhost:4000/v1/models/:modelId/original`
