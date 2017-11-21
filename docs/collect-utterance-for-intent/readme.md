# collect-utterance-for-intent API

* [Introduction](#introduction)
* [Create a Task](#create-a-task)
* [Get a Result](#get-a-result)
* [Delete Task when Task is complete](#delete-task-when-task-is-complete)
* [Get started with collect-utterance-for-intent](#get-started-with-collect-utterance-for-intent)


## Introduction
The `collect-utterance-for-intent` API enables you to ask Amazon Mechanical Turk (MTurk) Workers to provide a phrase or sentence given a context and intent. This is useful in the field of natural language processing, for example when training a chat bot understand the various ways people might ask for the same thing. 

The API takes as input two strings of text, the context and intent, both max length 400 characters, and returns an JSON array indicating utterances provided by the Worker.  When you call this API, it uses your MTurk Requester account and AWS account to automatically create HITs for MTurk Workers to complete, and automatically processes answers from Workers and returns the result.

### Example Input:

```Python
{
  "input": {
	"context": "They received a bill that is higher than expected.", 
	"intent": "They want to get bill reduced to expected amount."
   }
}
```

### Example Result:

```Python
{
  "result": {
    "utterances": [{"text":"Hi, I received my bill today, and I had a few questions about the charges."}]
  }
}
```

Here’s how the input might be presented in a HIT to Workers on MTurk.

![Worker Preview of HIT](https://s3-us-west-2.amazonaws.com/mturk-sample-tasks/collect-utterance-text-HIT-example.png "Worker Preview of HIT")

---
All of the single purpose APIs follow the same REST pattern as described in our [REST API reference]. They require a function name indicating which API you want to call, and a unique Task name used to represent your input and the corresponding result. 

`collect-utterance-for-intent-test` is a version of the `collect-utterance-for-intent` API meant for testing your integration. It has the same interface as the production version, but Tasks always complete quickly with mock results.  HITs are not created on MTurk so you don’t need to pay Workers for rewards.

> Note: This will return a random mock result when you call `get_task` with `collect-utterance-for-intent-test`

The rest of this article provides instructions on using the `collect-utterance-for-intent` API using the MTurk Python Client. 
[Download Sample Python code] for `collect-utterance-for-intent-test`


## Create a Task
This Python sample calls the `collect-utterance-for-intent` API using [Boto3] and the [MTurk Python client]. 

Tasks are created by calling `put_task`. For more details for how to use `put_task`, check out [Instructions for creating your first Task for a MTurk single purpose API].

The body of a `put_task` request looks like this:

```
{
  "input": {
    "context": <context of conversation utterance is requested>,
    "intent": <intended meaning of requested utterance>
  }
}
```

##### Input structure

| Name | Description | Type | Required | 
| ---- | ----------- | ---- | :------: |
| "context" | context of conversation utterance is requested| String (between 1-400 characters) | Yes|
| "intent" | intended meaning of requested utterance | String (between 1-400 characters) | Yes|


#### Example request

```
context = 'They recieved a bill that is higher than expected'
intent = 'They want to get bill reduced to expected amount'

put_result = crowd_client.put_task('collect-utterance-for-intent', 'my-task-name',                         
                             {'context': context, 'intent': intent })

```

#### Example response

```
{
  "taskName": "my-task-name",
  "input": {
    "context": "They received a bill that is higher than expected.",
    "intent": "They want to get bill reduced to expected amount."}
  },
  "problemDetails": null,
  "state": "processing",
  "result": null
}
```



## Get a Result
This Python sample calls the `collect-utterance-for-intent` API using [Boto3] and the [MTurk Python client]. 

After creating a Task, users can call the `get_task` to poll its current state. 

> Remember it can take some time for a Worker to complete a Task, so you might have to wait before getting a result. 

For more details for how to use `get_task`, check out [Instructions for creating your first Task for a MTurk single purpose API].

#### Example request

```
client.get_task('collect-utterance-for-intent', 'my-task-name')
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
| ---- | ----- | ---- | 
| "utterances" | list of responses from Workers | Array of JSON Objects | 
| "text" | single response from a Worker | String | 

#### Example response

Example response for a successful Task:

```
{
  "taskName": "my-task-name",
  "input": {
    "context": "They received a bill that is higher than expected.",
    "intent": "They want to get bill reduced to expected amount."}
  },
  "problemDetails": null,
  "state": "completed",
  "result": {"utterances": [{
    "text": "Hi, I received my bill today, and I had a few questions about the charges." }]
}
```

Example response for a failed Task:

```
{
  "taskName": "my-task-name",
  "input": {
    "context": "They received a bill that is higher than expected.",
    "intent": "They want to get bill reduced to expected amount."}
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
This Python sample calls the `collect-utterance-for-intent` API using [Boto3] and the [MTurk Python client]. 

A user can optionally delete a finished Task (one whose state is either “completed” or “failed”). After you delete a Task, you can reuse the Task name for a future Task.

>Note: Deleting a Task that is still being processed (i.e., whose state is “processing”) is not allowed.

For more details for how to use `delete_task`, check out [Instructions for creating your first Task for a MTurk single purpose API].

#### Example Input

```
client.delete_task('collect-utterance-for-intent', 'my-task-name')
```

#### Example Output

A successful request for the `delete_task` operation returns with no errors and an empty body.

---

## Get Started with collect-utterance-for-intent

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

[Download Sample Python code]: https://gist.github.com/AmazonMTurk/556bcefaab82f972797a9770915c52be#file-collect-utterance-for-intent-test-py

[please contact our product team]: mailto:mturk-requester-preview@amazon.com
[on GitHub]: https://github.com/awslabs/mturk-crowd-beta-client-python

