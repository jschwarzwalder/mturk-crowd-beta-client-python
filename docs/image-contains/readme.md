# image-contains API

* [Introduction](#introduction)
* [Create a Task](#create-a-task)
* [Get a Result](#get-a-result)
* [Delete Task when Task is complete](#delete-task-when-task-is-complete)
* [Get started with image-contains](#get-started-with-image-contains)

## Introduction
This `image-contains` API enables you to determine whether an image contains specific objects using Amazon Mechanical Turk (MTurk). This is useful in the field of computer vision, for example, data scientists commonly need to train machine learning models to detect specific objects in images, such as faces or cars.

The API takes as input an image URL and a type of object, then returns true or false if that object is detected in the image. When you call this API, it uses your MTurk Requester account and AWS account to automatically create HITs for Workers on MTurk to complete, and automatically processes answers from Workers and returns the result.


### Example request:

```python
{
  "input": {
      "image": {"url": "https://www.mturk.com/media/butterbean.jpg"}, 
      "target": {"label": "dog"} 
   }
}
```

#### Image Preview

| "image" | 
| -------- |  
|![Image1](https://www.mturk.com/media/butterbean.jpg "Preview of Image") 

> Note you can host images on [Amazon S3] if you do not have your own server.

### Example Result:

```python
{
  "result": {
    "containsTarget": True
  }
}
```

Here’s how the images might be presented to Workers on MTurk.

![Worker Preview of HIT](https://s3-us-west-2.amazonaws.com/mturk-sample-tasks/image-contains-HIT-example.jpeg "Worker Preview of HIT")

The API implements an adjudication strategy based on Worker agreement and returns results only when it reaches confidence.

---
All of the single purpose APIs follow the same REST pattern as described in our [REST API reference]. They require a function name indicating which API you want to call, and a unique Task name used to represent your input and the corresponding result. 

`image-contains-test` is a version of the `image-contains` API meant for testing your integration. It has the same interface as the production version, but Tasks always complete quickly with mock results.  HITs are not created on MTurk, so you don’t need to pay Workers for rewards.

> Note: This will return a random mock result when you call `get_task` with `image-contains-test`.

The rest of this article provides instructions on using the `image-contains` API using the MTurk Python Client.
[Download Sample Python code] for `image-contains-test`.


## Create a Task
This Python sample calls the `image-contains` API using [Boto3] and the [MTurk Python client]. 

Tasks are created by calling `put_task`. For more details for how to use `put_task`, check out [Instructions for creating your first Task for a MTurk single purpose API].

> Note: Before you create your Task, make sure you try to view the image using the URL in your browser to ensure it display correctly.

The body of a `put_task` request looks like this:

```
{
  "input": {
    "image": {"url": "<image you want analyzed>"},
    "target": {"label": "<the type of thing you're looking for>"}
  }
}
```

##### Input structure

| Name | Description | Type | Required | 
| ---- | ----------- | ---- | :------: |
|"image" | image you want to be analyzed | JSON Object | Yes|
|"url" | link to image hosted online (<= 400 characters) | String  | Yes|
|"target" | the target object Workers are asked to detect  | JSON Object | Yes|
|"label" | the name of the target object Workers are asked to detect | String | Yes|

> Note you can host images on [Amazon S3] if you do not have your own server.

#### Example request

```
image_url = 'https://www.mturk.com/media/butterbean.jpg'

label = 'dog'

put_result = crowd_client.put_task('image-contains', 'my-task-name',                         
                              {'image': {'url': image_url}, 'target': {'label': label} })
```

#### Example response

```
{
  "taskName": "my-task-name",
   "input": {
      "image": {"url": "https://www.mturk.com/media/butterbean.jpg"}, 
      "target": {"label": "dog"} 
   },
  "problemDetails": null,
  "state": "processing",
  "result": null
}
```



## Get a Result
This Python sample calls the `image-contains` API using [Boto3] and the [MTurk Python client]. 

After creating a Task, users can call the `get_task` to poll its current state. 

> Remember it can take some time for a Worker to complete a Task, so you might have to wait before getting a result. 

For more details for how to use `get_task`, check out [Instructions for creating your first Task for a MTurk single purpose API].

#### Example request

```
client.get_task('image-contains', 'my-task-name')
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
|"containsTarget" | indication whether object is detected in image.   | Boolean |

#### Example response

Example response for a successful Task:

```
{
  "taskName": "my-task-name",
  "input": {
      "image": {"url": "https://www.mturk.com/media/butterbean.jpg"}, 
      "target": {"label": "dog"} 
   },
  "problemDetails": null,
  "state": "completed",
  "result": {
      "containsTarget": True
  }
}
```

Example response for a failed Task:

```
{
  "taskName": "my-task-name",
   "input": {
      "image": {"url": "https://www.mturk.com/media/butterbean.jpg"}, 
      "target": {"label": "dog"} 
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
This Python sample calls the `image-contains` API using [Boto3] and the [MTurk Python client]. 

A user can optionally delete a finished Task (one whose state is either “completed” or “failed”). After you delete a Task, you can reuse the Task name for a future Task.

>Note: Deleting a Task that is still being processed (i.e., whose state is “processing”) is not allowed.

For more details for how to use `delete_task`, check out [Instructions for creating your first Task for a MTurk single purpose API].

#### Example Input

```
client.delete_task('image-contains', 'my-task-name')
```

#### Example Output

A successful request for the `delete_task` operation returns with no errors and an empty body.

---

## Get started with image-contains

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

[Download Sample Python code]: https://gist.github.com/AmazonMTurk/b03ff7750c8e51f6ac2064b63b1c7c81#file-image-contains-test-py

[please contact our product team]: mailto:mturk-requester-preview@amazon.com
[on GitHub]: https://github.com/awslabs/mturk-crowd-beta-client-python

[Amazon S3]: http://aws.amazon.com/s3
