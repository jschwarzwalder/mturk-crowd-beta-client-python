# coreference-resolution API

* [Introduction](#introduction)
* [Create a Task](#create-a-task)
* [Get a Result](#get-a-result)
* [Delete Task when Task is complete](#delete-task-when-task-is-complete)
* [Get started with coreference-resolution](#get-started-with-coreference-resolution)

## Introduction
The `coreference-resolution` API enables you to find expressions that refer to the same entity within a piece of text using Amazon Mechanical Turk (MTurk). This is useful in the field of natural language processing, for example when generating document summaries. 

The API takes as input of a string of text to be analyzed, max length 1,600 characters, and returns an array of strings indicating expressions that refer to the same entity within the input text.  When you call this API, it uses your MTurk Requester account and AWS account to automatically create HITs for Workers on MTurk to complete, and automatically processes answers from Workers and returns the result.

### Example Input:

```Python
{
  "input": {
    "text": "Megan said \"I voted for Hilary because she better aligns with my values"
   }
}
```

### Example Result:

```Python
{
  "result": {"coreferences": {
      ["references": [
                {"span":{"endIndex": 6, "startIndex": 0, "text": "Megan"}}, 
                {"span":{"endIndex": 14, "startIndex": 12, "text": "I" }}, 
                {"span":{"endIndex": 65, "startIndex": 62, "text": "my"}} 
                
      ], "references": [
                {"span":{"endIndex": 31, "startIndex": 24, "text": "Hilary" }},
                {"span":{"endIndex": 43, "startIndex": 39, "text": "she"}}
      ]
    ] }
  }
}
```

Here’s how the input might be presented in a HIT to Workers on MTurk.

![Worker Preview of HIT](https://s3-us-west-2.amazonaws.com/mturk-sample-tasks/co-reference-resolution-HIT-example.png "Worker Preview of HIT")

---

All of the single purpose APIs follow the same REST pattern as described in our [REST API reference]. They require a function name indicating which API you want to call, and a unique Task name used to represent your input and the corresponding result.

`coreference-resolution-test` is a version of the `coreference-resolution` API meant for testing your integration. It has the same interface as the production version, but Tasks always complete quickly with mock results.  HITs are not created on MTurk so you don’t need to pay Workers for rewards.

> Note: This will return a random mock result when you call `get_task` with `coreference-resolution-test`


The rest of this article provides instructions on using the `coreference-resolution` API using the MTurk Python Client. 
[Download Sample Python code] for `coreference-resolution -test`


## Create a Task
This Python sample calls the `coreference-resolution` API using [Boto3] and the [MTurk Python client]. 

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
|"text" | text you want analyzed | String (between 1-1,600 characters) | Yes|


#### Example request

```
text = 'Megan said "I voted for Hilary because she better aligns with my values"'

put_result = crowd_client.put_task('coreference-resolution', 'my-task-name',                         
                             {'text': text})

```

#### Example response

```
{
  "taskName": "my-task-name",
  "input": {
    "text": "Megan said \"I voted for Hilary because she better aligns with my values"
  },
  "problemDetails": null,
  "state": "processing",
  "result": null
}
```



## Get a Result
This Python sample calls the `coreference-resolution` API using [Boto3] and the [MTurk Python client]. 

After creating a Task, users can call the `get_task` to poll its current state. 

> Remember it can take some time for a Worker to complete a Task, so you might have to wait before getting a result. 


For more details for how to use `get_task`, check out [Instructions for creating your first Task for a MTurk single purpose API].

#### Example request

```
client.get_task('coreference-resolution', 'my-task-name')
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
| "coreferences" | list of references in provided string | Array of JSON Objects| 
| "references" | list of span JSON objects which indicates the location of a single reference | Array of JSON Objects| 
| "span" | an extracted reference, along with its position in the input string  | JSON Object|
| "endIndex" | position of the end of selected reference | Integer | 
| "startIndex" | position of the start of selected reference | Integer | 
| "text" | text of selected reference | String |

#### Example response

Example response for a successful Task:

```
{
  "taskName": "my-task-name",
  "input": {
    "text": "Megan said \"I voted for Hilary because she better aligns with my values"
  },
  "problemDetails": null,
  "state": "completed",
  {
  "result": {"coreferences": {
      ["references": [
                {"span":{"endIndex": 6, "startIndex": 0, "text": "Megan"}}, 
                {"span":{"endIndex": 14, "startIndex": 12, "text": "I" }}, 
                {"span":{"endIndex": 65, "startIndex": 62, "text": "my"}} 
                
      ], "references": [
                {"span":{"endIndex": 31, "startIndex": 24, "text": "Hilary" }},
                {"span":{"endIndex": 43, "startIndex": 39, "text": "she"}}
      ]
    ] }
  }
}


```

Example response for a failed Task:

```
{
  "taskName": "my-task-name",
  "input": {
    "text": "Megan said \"I voted for Hilary because she better aligns with my values"
  },
  "problemDetails": {
    "code": "Expired",
    "message": "Not enough Workers provided answers for your task within the allotted time."
  },
  "state": "failed",
  "result": null
}
```

> Note: If Workers do not find any coreferences in the provided input string, "state" will still return as complete, but in the result, "coreferences" will be an empty array. 


## Delete Task when Task is complete
This Python sample calls the `coreference-resolution` API using [Boto3] and the [MTurk Python client]. 

A user can optionally delete a finished Task (one whose state is either “completed” or “failed”). After you delete a Task, you can reuse the Task name for a future Task.

>Note: Deleting a Task that is still being processed (i.e., whose state is “processing”) is not allowed.

For more details for how to use `delete_task`, check out [Instructions for creating your first Task for a MTurk single purpose API].

#### Example Input

```
client.delete_task('coreference-resolution', 'my-task-name')
```

#### Example Output

A successful request for the `delete_task` operation returns with no errors and an empty body.

---

## Get Started with coreference-resolution

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


[Download Sample Python code]: https://gist.github.com/AmazonMTurk/b5a45838611f223efbfd662103301c69#file-coreference-resolution-test-py

[please contact our product team]: mailto:mturk-requester-preview@amazon.com
[on GitHub]: https://github.com/awslabs/mturk-crowd-beta-client-python

