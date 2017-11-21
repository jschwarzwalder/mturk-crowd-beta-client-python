# key-phrase-extraction API

* [Introduction](#introduction)
* [Create a Task](#create-a-task)
* [Get a Result](#get-a-result)
* [Delete Task when Task is complete](#delete-task-when-task-is-complete)
* [Get started with key-phrase-extraction](#get-started-with-key-phrase-extraction)

## Introduction
The `key-phrase-extraction` API enables you to identify the essential words and phrases from a paragraph of text which is useful in use cases such as document categorization, summarizing and comparing the similarity of documents.

The API takes as an input of a string of text to be analyzed, max length 1,600 characters, and returns an array of JSON Objects indicating key phrases along with their location in the input string starting at index 0.  When you call this API, it uses your MTurk Requester account and AWS account to automatically create HITs for Workers on MTurk to complete, and automatically processes answers from Workers and returns the result.

### Example Input:

```
{
  "input": {
    "text": "I love this album. I especially like Peter's version of  \"The
        Parting Glass\" and his collaboration with Jackie Evancho on \"Hallelujah\"."
  }
}
```

### Example Result:

```Python
{
  "result": {
    {"keyPhrases": [
		{'span': 
			{'text':'love this album', 
			'startIndex': 2,
			'endIndex': 17}
		}, 
		{'span': 
			{'text':'collaboration', 
			'startIndex': 84 , 
			'endIndex': 97}
		}
	] }
  }
}
```

Here’s how the input might be presented in a HIT to Workers on MTurk.

![Worker Preview of HIT](https://s3-us-west-2.amazonaws.com/mturk-sample-tasks/key-phrase-extraction-HIT-example.png "Worker Preview of HIT")

---

All of the single purpose APIs follow the same REST pattern as described in our [REST API reference]. They require a function name indicating which API you want to call, and a unique Task name used to represent your input and the corresponding result. 

`key-phrase-extraction-test` is a version of the `key-phrase-extraction` API meant for testing your integration. It has the same interface as the production version, but Tasks always complete quickly with mock results.  HITs are not created on MTurk, so you don’t need to pay Workers for rewards.

> Note: This will return a random mock result when you call `get_task` with `key-phrase-extraction-test`

The rest of this article provides instructions on using the `key-phrase-extraction` API using the MTurk Python Client.
[Download Sample Python code] for `key-phrase-extraction-test`


## Create a Task
This Python sample calls the `key-phrase-extraction` API using [Boto3] and the [MTurk Python client]. 

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
text = 'I love this album. I especially like Peter\'s version of 
		"The Parting Glass" and his collaboration with Jackie Evancho on "Hallelujah".'

put_result = crowd_client.put_task('key-phrase-extraction', 'my-task-name',                         
                             {'text': text})

```

#### Example response

```
{
  "taskName": "my-task-name",
  "input": {
    "text": "I love this album. I especially like Peter's version of \"The
	   Parting Glass\" and his collaboration with Jackie Evancho on \"Hallelujah\"."
  },
  "problemDetails": null,
  "state": "processing",
  "result": null
}
```



## Get a Result
This Python sample calls the `key-phrase-extraction` API using [Boto3] and the [MTurk Python client]. 

After creating a Task, users can call the `get_task` to poll its current state. 

> Remember it can take some time for a Worker to complete a Task, so you might have to wait before getting a result. 

For more details for how to use `get_task`, check out [Instructions for creating your first Task for a MTurk single purpose API].

#### Example request

```
client.get_task('key-phrase-extraction', 'my-task-name')
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
| "keyPhrases" | the key phrases, as specified by a list of spans, extracted from the input string | Array of JSON Objects| 
| "span" | an extracted key phrase, along with its position in the input string  | JSON Object|
| "text" | text of selected key phrase | String |
| "startIndex" | position of the start of selected key phrase | Integer | 
| "endIndex" | position of the end of selected key phrase | Integer | 

#### Example response

Example response for a successful Task:

```
{
  "taskName": "my-task-name",
  "input": {
    "text": "I love this album. I especially like Peter's version of 
			\"The Parting Glass\" and his collaboration with Jackie Evancho on \"Hallelujah\"."
  },
  "problemDetails": null,
  "state": "completed",
  "result": {
      {"keyPhrases": [
		{'span': 
			{'text':'love this album', 
			'startIndex': 2,
			'endIndex': 17}
		}, 
		{'span': 
			{'text':'collaboration', 
			'startIndex': 84 , 
			'endIndex': 97}
		}
	] }
  }
}
```

Example response for a failed Task:

```
{
  "taskName": "my-task-name",
  "input": {
    "text": "I love this album. I especially like Peter's version of 
			\"The Parting Glass\" and his collaboration with Jackie Evancho on \"Hallelujah\"."
  },
  "problemDetails": {
    "code": "Expired",
    "message": "Not enough Workers provided answers for your task within the allotted time."
  },
  "state": "failed",
  "result": null
}
```

> Note: If Workers do not find any key phrases in the provided input string, "state" will still return as complete, but in the result, "keyPhrases" will be an empty array. 


## Delete Task when Task is complete
This Python sample calls the `key-phrase-extraction` API using [Boto3] and the [MTurk Python client]. 

A user can optionally delete a finished Task (one whose state is either “completed” or “failed”). After you delete a Task, you can reuse the Task name for a future Task.

>Note: Deleting a Task that is still being processed (i.e., whose state is “processing”) is not allowed.

For more details for how to use `delete_task`, check out [Instructions for creating your first Task for a MTurk single purpose API].

#### Example Input

```
client.delete_task('key-phrase-extraction', 'my-task-name')
```

#### Example Output

A successful request for the `delete_task` operation returns with no errors and an empty body.

---

## Get started with key-phrase-extraction

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


[Download Sample Python code]: https://gist.github.com/AmazonMTurk/2bcb0ad7c3526564b88d790087541ee7#file-key-phrase-extraction-test-py

[please contact our product team]: mailto:mturk-requester-preview@amazon.com
[on GitHub]: https://github.com/awslabs/mturk-crowd-beta-client-python

