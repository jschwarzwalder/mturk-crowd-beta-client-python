# bounding-box API

* [Introduction](#introduction)
* [Create a Task](#create-a-task)
* [Get a Result](#get-a-result)
* [Delete Task when Task is complete](#delete-task-when-task-is-complete)
* [Get started with bounding-box](#get-started-with-bounding-box)

## Introduction
The `bounding-box` API enables you to detect objects you’re looking for and returns the location of these objects in images you provide using Amazon Mechanical Turk (MTurk). The location is indicated by a bounding box around each object. This is useful in the field of computer vision, such as annotating a feed from a camera, e.g., single frame from a video, and detecting various object types such as pedestrians, cars and obstacles.

The API takes as input a URL for an image to be analyzed and an array of labels indicating objects to be identified. You can specify multiple objects to look for, such as cars and pedestrians. This API returns an array of bounding boxes for each labeled object in the image. When you call this API, it uses your MTurk Requester account and AWS account to automatically create HITs for Workers on MTurk to complete, and automatically processes answers from Workers and returns the result.

### Example Input:

```Python
{
   "input": {
      "image": {"url": "https://s3-us-west-2.amazonaws...box-hit-example.jpg"}, 
      "targets": [{"label": "car"}, 
            {"label": "pedestrian"}, 
            {"label": "tram"}
      ]
   }
}
```

#### Image Preview

| "image" | 
| -------- |  
|![Image1](https://s3-us-west-2.amazonaws.com/mturk-docs-media/box-hit-example.jpg "Preview of Image") 

> Note you can host images on [Amazon S3] if you do not have your own server.


### Example Result:

```Python
{
  "result": {
    "annotations": [
        {"boundingBox": {
            "height": 77, 
            "left": 819, 
            "top": 498, 
            "width": 70 }, 
         "label": "car"
        },  {"boundingBox": {
            "height": 437, 
            "left": 262, 
            "top": 311, 
            "width": 368 }, 
         "label": "tram"
        }, {"boundingBox": {
            "height": 64, 
            "left": 633, 
            "top": 518, 
            "width": 22 }, 
         "label": "pedestrian"
        },  {"boundingBox": {
            "height": 203, 
            "left": 224, 
            "top": 490, 
            "width": 67 }, 
         "label": "pedestrian"
        }
    ]
  }
}
```

Here’s how the input might be presented in a HIT to Workers on MTurk.

![Worker Preview of HIT](https://s3-us-west-2.amazonaws.com/mturk-sample-tasks/bounding-box-HIT-example.png "Worker Preview of HIT")

---
All of the single purpose APIs follow the same REST pattern as described in our [REST API reference]. They require a function name indicating which API you want to call, and a unique Task name used to represent your input and the corresponding result. 

`bounding-box-test` is a version of the `bounding-box` API meant for testing your integration. It has the same interface as the production version, but Tasks always complete quickly with mock results.  HITs are not created on MTurk, so you don’t need to pay Workers for rewards.

> Note: This will return a random mock result when you call `get_task` with `bounding-box-test`.

The rest of this article provides instructions on using the `bounding-box` API using the MTurk Python Client.
[Download Sample Python code] for `bounding-box-test`.


## Create a Task
This Python sample calls the `bounding-box` API using [Boto3] and the [MTurk Python client]. 

Tasks are created by calling `put_task`. For more details on how to use `put_task`, check out [Instructions for creating your first Task for a MTurk single purpose API].

> Note: Before you create your Task, make sure you try to view the image using the URL in your browser to ensure it displays correctly.


The body of a `put_task` request looks like this:

```
{
  "input": {
     "image": {"url": "<image you want analyzed>"},
     "targets": [{"label": "<object worker should identify>"}]
  }
}
```

##### Input structure

| Name | Description | Type | Required | 
| ---- | ----------- | ---- | :------: |
|"image" | image you want to be analyzed | JSON Object | Yes|
|"url" | link to image hosted online (<= 400 characters) | String  | Yes|
|"targets" | up to ten target objects Workers are asked to detect  | Array of JSON Object | Yes|
|"label" | the name of the target object Workers are asked to detect (<= 100 characters) | String | Yes|

> Note you can host images on [Amazon S3] if you do not have your own server.

#### Example request

```
image_url = 'https://s3-us-west-2.amazonaws.com/mturk-sample-media/box-hit-example.jpg'

label1 = 'car'
label2 = 'pedestrian'
label3 = 'tram'


put_result = crowd_client.put_task('bounding-box', 'my-task-name',                         
                              {'image': {'url': image_url}, 
                              "labels": [{"label": label1}, 
			                              {"label": label2}, 
			                              {"label": label3}]})


```

#### Example response

```
{
  "taskName": "my-task-name",
  "input": {
       "image": {"url": "https://s3-us-west-2.amazonaws...box-hit-example.jpg"}, 
      "target": {"label": "car"}
   },
  "problemDetails": null,
  "state": "processing",
  "result": null
}
```



## Get a Result
This Python sample calls the `bounding-box` API using [Boto3] and the [MTurk Python client]. 

After creating a Task, users can call the `get_task` to poll its current state. 

> Remember it can take some time for a Worker to complete a Task, so you might have to wait before getting a result. 

For more details for how to use `get_task`, check out [Instructions for creating your first Task for a MTurk single purpose API].

#### Example request

```
client.get_task('bounding-box', 'my-task-name')
```

#### Response structure:

| Name | Description | Type | 
| ---- | ----------- | ---- |
|"taskName" | user-provided Task name  | String|
|"input" | input provided by user when Task was created  | JSON Object|
|"problemDetails" | if the "state" is "failed" - details about a failed Task | JSON Object or null|
|"state" | current Task state - one of "processing", "completed" or "failed" | String|
|"result" | if the "state" is "completed" - the results of the completed Task  | JSON Object or null|



#### Result structure:

| Name | Value | Type | 
| ---- | ----------- | ---- |
|"annotations" | list of targets selected by Workers   | Array of JSON Objects |
|"boundingBox" | location of a single target and label of target  | JSON Object |
|"height" | size of vertical height of selection box in pixels  | Integer |
|"left" | x coordinate for top left corner of the bounding box  | Integer |
|"top" | y coordinate for top left corner of the bounding box  | Integer |
|"width" | size of horizontal width of selection box in pixels  | Integer |
|"label" | the name of the target object Workers are asked to detect | String | Yes|

#### Example response

Example response for a successful Task:

```
{
  "taskName": "my-task-name",
  "input": {
       "image": {"url": "https://s3-us-west-2.amazonaws...box-hit-example.jpg"}, 
      "target": {"label": "car"}
   },
  "problemDetails": null,
  "state": "completed",
  "result": {"annotations": 
             [{"boundingBox": {
            "height": 77, "left": 819, 
            "top": 498, "width": 70 }, 
         "label": "car"
        },  {"boundingBox": {
            "height": 437,"left": 262, 
            "top": 311, "width": 368 }, 
         "label": "tram"
        }, {"boundingBox": {
            "height": 64, "left": 633, 
            "top": 518,"width": 22 }, 
         "label": "pedestrian"
        },  {"boundingBox": {
            "height": 203, "left": 224, 
            "top": 490,  "width": 67 }, 
         "label": "pedestrian"
        }]}
}
```

Example response for a failed Task:

```
{
  "taskName": "my-task-name",
  "input": {
       "image": {"url": "https://s3-us-west-2.amazonaws...box-hit-example.jpg"}, 
      "target": {"label": "car"}
   },
  "problemDetails": {
    "code": "Expired",
    "message": "Not enough Workers provided answers for your task within the allotted time."
  },
  "state": "failed",
  "result": null
}
```
## Delete Task when Task is complete
This Python sample calls the `bounding-box` API using [Boto3] and the [MTurk Python client]. 

A user can optionally delete a finished Task (one whose state is either “completed” or “failed”). After you delete a Task, you can reuse the Task name for a future Task.

>Note: Deleting a Task that is still being processed (i.e., whose state is “processing”) is not allowed.

For more details on how to use `delete_task`, check out [Instructions for creating your first Task for a MTurk single purpose API].

#### Example Input

```
client.delete_task('bounding-box', 'my-task-name')
```

#### Example Output

A successful request for the `delete_task` operation returns with no errors and an empty body.

---

## Get started with bounding-box

1. [Set up your Amazon Mechanical Turk (MTurk) Requester account and AWS account]
1. [Set up permissions to call an MTurk single purpose API]
1. [Instructions for creating your first Task for a MTurk single purpose API]

MTurk has released several APIs for common use cases, [click here to see a list of all the available APIs].

If you have any questions or feedback, such as methods you wish our client supported, [please contact our product team], or submit a pull request [on GitHub] adding additional functionality to our client.

[REST API reference]: ../REST.md
[Set up your Amazon Mechanical Turk (MTurk) Requester account and AWS account]: ../step_0_setup_accounts.md
[Set up permissions to call an MTurk single purpose API]: ../step_1_setup_aws_user.md
[Install Python client and create your first Tasks with MTurk API]: ../step_2_first_task.md
[Instructions for creating your first Task for a MTurk single purpose API]: ../step_2_first_task.md
[click here to see a list of all the available APIs]: ../../readme.md#what-apis-are-available

[Boto3]: https://boto3.readthedocs.io/en/latest/guide/quickstart.html#installation
[MTurk Python client]: https://github.com/awslabs/mturk-crowd-beta-client-python


[Download Sample Python code]: https://gist.github.com/AmazonMTurk/8481d72333a7e216bd9392d2d6bacf8d#file-bounding-box-test-py

[please contact our product team]: mailto:mturk-requester-preview@amazon.com
[on GitHub]: https://github.com/awslabs/mturk-crowd-beta-client-python

[Amazon S3]: http://aws.amazon.com/s3
