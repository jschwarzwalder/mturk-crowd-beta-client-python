# image-similarity API

* [Introduction](#introduction)
* [Create a Task](#create-a-task)
* [Get a Result](#get-a-result)
* [Delete Task when Task is complete](#delete-task-when-task-is-complete)
* [Get started with image-similarity](#get-started-with-image-similarity)

## Introduction
This `image-similarity` API enables you to compare two images and determine how visually similar they’re to each other using Amazon Mechanical Turk (MTurk). This is useful in the field of computer vision, for example when building a visual search that returns similar looking products. 

The API takes as input the URLs of the two images to be analyzed and returns a score between 0 and 1 indicating how similar the images are to each other with 1 being the most similar. When you call this API, it uses your MTurk Requester account and AWS account to automatically create HITs for Workers on MTurk to complete, and automatically processes answers from Workers and returns the result.


### Example request:

```python
{
  "input": {
    "image1": {"url":"https://requester.mturk.com/assets/lucy.jpg"},
    "image2": {"url":"https://requester.mturk.com/assets/gold-finding-puppy.jpg"}
   }
}
```

#### Image Preview

| "image1" | "image2" |
| -------- | -------- | 
|![Image1](https://s3-us-west-2.amazonaws.com/mturk-sample-media/images-to-compare/image-similarity-a1.jpg "Preview of Image1") | ![Image2](https://s3-us-west-2.amazonaws.com/mturk-sample-media/images-to-compare/image-similarity-g1.png "Preview of Image2") |

> Note you can host images on [Amazon S3] if you do not have your own server.

### Example Result:

```python
{
  "result": {
    "similarityScore": 0.5
  }
}
```

Here’s how the images might be presented to Workers on MTurk.

![Worker Preview of HIT](https://cdn-images-1.medium.com/max/800/1*S268GrGji75KrQmfW62LNA.png "Worker Preview of HIT")

The API implements an adjudication strategy based on Worker agreement and returns results only when it reaches confidence.

---
All of the single purpose APIs follow the same REST pattern as described in our [REST API reference]. They require a function name indicating which API you want to call, and a unique Task name used to represent your input and the corresponding result. 

`image-similarity-test` is a version of the `image-similarity` API meant for testing your integration. It has the same interface as the production version, but Tasks always complete quickly with mock results.  HITs are not created on MTurk, so you don’t need to pay Workers for rewards.

> Note: This will return a random mock result when you call `get_task` with `image-similarity-test`.

The rest of this article provides instructions on using the `image-similarity` API using the MTurk Python Client.
[Download Sample Python code] for `image-similarity-test`.


## Create a Task
This Python sample calls the `image-similarity` API using [Boto3] and the [MTurk Python client]. 

Tasks are created by calling `put_task`. For more details for how to use `put_task`, check out [Instructions for creating your first Task for a MTurk single purpose API].

> Note Before you create your Task, make sure you try to view the images in your browser to ensure they display correctly for the Workers. 

The body of a `put_task` request looks like this:

```
{
  "input": {
    "image1": {"url": "<image you want analyzed>"},                 
    "image2": {"url": "<image you want analyzed>"} 
}
```

##### Input structure

| Name | Description | Type | Required | 
| ---- | ----------- | ---- | :------: |
|"image1" | first image you want analyzed | JSON Object | Yes|
|"image2" | second image you want analyzed | JSON Object | Yes|
|"url" | link to image hosted online (<= 400 characters) | String  | Yes|

> Note you can host images on [Amazon S3] if you do not have your own server.

#### Example request

```
image1 = 'https://s3-us-west-2.amazonaws.com/mturk-sample-media/images-to-compare/image-similarity-a1.jpg'
image2 = 'https://s3-us-west-2.amazonaws.com/mturk-sample-media/images-to-compare/image-similarity-g1.png'


put_result = crowd_client.put_task('image-similarity', 'my-task-name',                         
                              {'image1': {'url': image1}, 'image2': {'url': image2}})

```

#### Example response

```
{
  "taskName": "my-task-name",
   "input": {
    "image1": {"url":"https://requester.mturk.com/assets/lucy.jpg"},
    "image2": {"url":"https://requester.mturk.com/assets/gold-finding-puppy.jpg"}
   },
  "problemDetails": null,
  "state": "processing",
  "result": null
}
```



## Get a Result
This Python sample calls the `image-similarity` API using [Boto3] and the [MTurk Python client]. 

After creating a Task, users can call the `get_task` to poll its current state. 

> Remember it can take some time for a Worker to complete a Task, so you might have to wait before getting a result. 

For more details for how to use `get_task`, check out [Instructions for creating your first Task for a MTurk single purpose API].

#### Example request

```
client.get_task('image-similarity', 'my-task-name')
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
|"similarityScore" | a score between 0 and 1 indicating similarity of images with 1 being the most similar  | Integer |

#### Example response

Example response for a successful Task:

```
{
  "taskName": "my-task-name",
   "input": {
    "image1": {"url":"https://requester.mturk.com/assets/lucy.jpg"},
    "image2": {"url":"https://requester.mturk.com/assets/gold-finding-puppy.jpg"}
   },
  "problemDetails": null,
  "state": "completed",
  "result": {
    "similarityScore": 0.5
  }
}
```

Example response for a failed Task:

```
{
  "taskName": "my-task-name",
   "input": {
    "image1": {"url":"https://requester.mturk.com/assets/lucy.jpg"},
    "image2": {"url":"https://requester.mturk.com/assets/gold-finding-puppy.jpg"}
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
This Python sample calls the `image-similarity` API using [Boto3] and the [MTurk Python client]. 

A user can optionally delete a finished Task (one whose state is either “completed” or “failed”). After you delete a Task, you can reuse the Task name for a future Task.

>Note: Deleting a Task that is still being processed (i.e., whose state is “processing”) is not allowed.

For more details for how to use `delete_task`, check out [Instructions for creating your first Task for a MTurk single purpose API].

#### Example Input

```
client.delete_task('image-similarity', 'my-task-name')
```

#### Example Output

A successful request for the `delete_task` operation returns with no errors and an empty body.

---

## Get started with image-similarity

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


[Download Sample Python code]: https://gist.github.com/AmazonMTurk/c3a36eeea52c3a1076f3c09f80d40d85#file-image-similarity-test-py

[please contact our product team]: mailto:mturk-requester-preview@amazon.com
[on GitHub]: https://github.com/awslabs/mturk-crowd-beta-client-python

[Amazon S3]: http://aws.amazon.com/s3
