# sentiment-analysis API

* [Introduction](#introduction)
* [Create a Task](#create-a-task)
* [Get a Result](#get-a-result)
* [Delete Task when Task is complete](#delete-task-when-task-is-complete)
* [Get started with sentiment-analysis](#get-started-with-sentiment-analysis)

## Introduction
The `sentiment-analysis` API enables you to determine the sentiment of a piece of text using Amazon Mechanical Turk (MTurk). This is useful in the field of natural language processing, for example when analyzing large quantities of customer reviews. 

The API takes as input a string to be analyzed, max length 400 characters, and the result is one of _positive, negative, neutral or cannot determine_.  When you call this API, it uses your MTurk Requester account and AWS account to automatically create HITs for Workers on MTurk to complete, and automatically processes answers from Workers and returns the result.

### Example Input:

```Python
{
  "input": {
    "text": "Everything is wonderful."
   }
}
```

### Example Result:

```Python
{
  "result": {
    "sentiment": "positive" | "negative" | "neutral" | "cannot determine"
  }
}
```

Here’s how the input might be presented in a HIT to Workers on MTurk.

![Worker Preview of HIT](https://s3-us-west-2.amazonaws.com/mturk-sample-tasks/sentiment-analysis-HIT-example.png "Worker Preview of HIT")

The API implements an adjudication strategy based on Worker agreement and returns results only when it reaches confidence.

---
All of the single purpose APIs follow the same REST pattern as described in our [REST API reference]. They require a function name indicating which API you want to call, and a unique Task name used to represent your input and the corresponding result. 

`sentiment-analysis-test` is a version of the `sentiment-analysis` API meant for testing your integration. It has the same interface as the production version, but Tasks always complete quickly with mock results.  HITs are not created on MTurk so you don’t need to pay Workers for rewards.

> Note: This will return a random mock result when you call `get_task` with `sentiment-analysis-test`

The rest of this article provides instructions on using the `sentiment-analysis` API using the MTurk Python Client.
[Download Sample Python code] for `sentiment-analysis-test`


## Create a Task
This Python sample calls the `sentiment-analysis` API using [Boto3] and the [MTurk Python client]. 

Tasks are created by calling `put_task`. For more details for how to use `put_task`, check out [Instructions for creating your first Task for a MTurk single purpose API].

The body of a `put_task` request looks like this:

```
{
  "input": {
    "text": <text you want analyzed>
  }
}
```

##### Input structure

| Name | Description | Type | Required | 
| ---- | ----------- | ---- | :------: |
|"text" | text you want analyzed | String (between 1-400 characters) | Yes|


#### Example request

```
text = 'Everything is wonderful.'

put_result = crowd_client.put_task('sentiment-analysis', 'my-task-name',                         
                             {'text': text})

```

#### Example response

```
{
  "taskName": "my-task-name",
  "input": {
    "text": "Everything is wonderful!"
  },
  "problemDetails": null,
  "state": "processing",
  "result": null
}
```



## Get a Result
This Python sample calls the `sentiment-analysis` API using [Boto3] and the [MTurk Python client]. 

After creating a Task, users can call the `get_task` to poll its current state. 

> Remember it can take some time for a Worker to complete a Task, so you might have to wait before getting a result. 

For more details for how to use `get_task`, check out [Instructions for creating your first Task for a MTurk single purpose API].

#### Example request

```
client.get_task('sentiment-analysis', 'my-task-name')
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
|"sentiment" | one of *"positive", "negative", "neutral"* or *"cannot determine"*  | String |

#### Example response

Example response for a successful Task:

```
{
  "taskName": "my-task-name",
  "input": {
    "text": "Everything is wonderful!"
  },
  "problemDetails": null,
  "state": "completed",
  "result": {
    "sentiment": "positive"
  }
}
```

Example response for a failed Task:

```
{
  "taskName": "my-task-name",
  "input": {
    "text": "Everything is wonderful!"
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
This Python sample calls the `sentiment-analysis` API using [Boto3] and the [MTurk Python client]. 

A user can optionally delete a finished Task (one whose state is either “completed” or “failed”). After you delete a Task, you can reuse the Task name for a future Task.

>Note: Deleting a Task that is still being processed (i.e., whose state is “processing”) is not allowed.

For more details for how to use `delete_task`, check out [Instructions for creating your first Task for a MTurk single purpose API].

#### Example Input

```
client.delete_task('sentiment-analysis', 'my-task-name')
```

#### Example Output

A successful request for the `delete_task` operation returns with no errors and an empty body.

---

## Get started with sentiment-analysis

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


[Download Sample Python code]: https://gist.github.com/AmazonMTurk/a97a9d34dbe3246646989527f8439b90#file-sentiment-analysis-test-py

[please contact our product team]: mailto:mturk-requester-preview@amazon.com
[on GitHub]: https://github.com/awslabs/mturk-crowd-beta-client-python

