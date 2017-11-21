# intent-detection API

* [Introduction](#introduction)
* [Create a Task](#create-a-task)
* [Get a Result](#get-a-result)
* [Delete Task when Task is complete](#delete-task-when-task-is-complete)
* [Get started with intent-detection](#get-started-with-intent-detection)

## Introduction
The `intent-detection` API enables you to determine the intention of a person using Amazon Mechanical Turk (MTurk) based on what he or she is saying in text form. This is a common use case in chat bot training to understand what needs a human is trying to express.

The API takes as input a string of text, max length 400 characters, to be analyzed, and an array of possible intents between size 2 and 10. The API returns the intent that best matches the text based on MTurk Worker responses. When you call this API, it uses your MTurk Requester account and AWS account to automatically create HITs for Workers on MTurk to complete, and automatically processes answers from Workers and returns the result.

### Example Input:

```Python
{
  "input": {
     "text": "my son was stung by bees and I need to know if I need to go to the ER",
     "intents":[
        {"label": "Schedule an appointment", 
         "description": "Caller expresses an intent to visit with health care professional"
         "positiveExamples":  [
               {"text": "I need to make an appointment with Dr. Smith" },
               {"text": "Can the doctor see me today?"}], 
         "negativeExamples":  [
               {"text": "Do you sell burritos?" },
               {"text": "Do you accept credit cards?"}] },
        {"label": "Medical Record Request", 
         "description": "Caller expresses a desire to obtain copies of their medical history"
         "positiveExamples":  [
               {"text": "I need a copy of my kids’ immunization records" },
               {"text": "Can you fax my test results to my surgeon"}], 
         "negativeExamples":  [
              {"text": "I need to refill my prescription"},
               {"text": "Do you have my test results?"}] }]
   }
}
```

### Example Result:

```Python
{
  "result": {
    "matches": [{"label": "Schedule an appointment"}]
  }
}
```

Here’s how the input might be presented in a HIT to Workers on MTurk.

![Worker Preview of HIT](https://s3-us-west-2.amazonaws.com/mturk-sample-tasks/text-intent-detection-HIT-example.png "Worker Preview of HIT")

The API implements an adjudication strategy based on Worker agreement and returns results only when it reaches confidence.

---
All of the single purpose APIs follow the same REST pattern as described in our [REST API reference]. They require a function name indicating which API you want to call, and a unique Task name used to represent your input and the corresponding result. 

`intent-detection-test` is a version of the `intent-detection` API meant for testing your integration. It has the same interface as the production version, but Tasks always complete quickly with mock results.  HITs are not created on MTurk, so you don’t need to pay Workers for rewards.

> Note: This will return a random mock result when you call `get_task` with `intent-detection-test`.

The rest of this article provides instructions on using the `intent-detection` API using the MTurk Python Client.
[Download Sample Python code] for `intent-detection-test`.


## Create a Task
This Python sample calls the `intent-detection` API using [Boto3] and the [MTurk Python client]. 

Tasks are created by calling `put_task`. For more details for how to use `put_task`, check out [Instructions for creating your first Task for a MTurk single purpose API].

The body of a `put_task` request looks like this:

```
{
  "input": {
    "text": <text you want analyzed>,
    "intents": <array of possible intents>
  }
}
```

##### Input structure

| Name | Description | Type | Required | 
| ---- | ----------- | ---- | :------: |
|"text" | text you want analyzed | String (between 1-1,600 characters) | Yes |
|"intents" | up to ten user-supplied intents, each with a label, description, and examples | Array of JSON Objects | Yes |
|"label" | the label representing the name of a single intent  | String (between 1-100 characters) | Yes|
|"description" | clarifying details about the intent presented to Workers | String | No |
|"positiveExamples" | up to two examples that fit the intent | Array of JSON Objects | No |
|"negativeExamples" | up to two examples that do not fit the intent | Array of JSON Objects | No |


#### Example request

```
text = 'my son was stung by bees and I need to know if I need to go to the ER.'

intents = [
	{'label': 'Schedule an appointment', 
	'description': 'Caller expresses an intent to visit with health care professional',
	'positiveExamples':  [
		{'text': 'I need to make an appointment with Dr. Smith' },
		{'text': 'Can the doctor see me today?'}],
	'negativeExamples': [
		{'text':'Do you sell burritos?' },
		{'text': 'Do you accept credit cards?'}] } ,
	{'label': 'Medical Record Request', 
	'description': 'Caller expresses a desire to obtain copies of their medical history',
	'positiveExamples': [
		{'text':'I need a copy of my kids’ immunization records' },
		{'text': 'Can you fax my test results to my surgeon'}] ,
	'negativeExamples': [
		{'text':'I need to refill my prescription' },
		{'text': 'Do you have my test results?'} ] }  ]

put_result = crowd_client.put_task('intent-detection', 'my-task-name',                         
                             {'text': text, 'intents': intents })

```

#### Example response

```
{
  "taskName": "my-task-name",
    "input": {
     "text": "my son was stung by bees and I need to know if I need to go to the ER",
     "intents":[
        {"label": "Schedule an appointment", 
         "description": "Caller expresses an intent to visit with health care professional"
         "positiveExamples":  [
               {"text": "I need to make an appointment with Dr. Smith" },
               {"text": "Can the doctor see me today?"}], 
         "negativeExamples":  [
               {"text": "Do you sell burritos?" },
               {"text": "Do you accept credit cards?"}] },
        {"label": "Medical Record Request", 
         "description": "Caller expresses a desire to obtain copies of their medical history"
         "positiveExamples":  [
               {"text": "I need a copy of my kids’ immunization records" },
               {"text": "Can you fax my test results to my surgeon"}], 
         "negativeExamples":  [
              {"text": "I need to refill my prescription"},
               {"text": "Do you have my test results?"}] }]
   },
  "problemDetails": null,
  "state": "processing",
  "result": null
}
```



## Get a Result
This Python sample calls the `intent-detection` API using [Boto3] and the [MTurk Python client]. 

After creating a Task, users can call the `get_task` to poll its current state. 

> Remember it can take some time for a Worker to complete a Task, so you might have to wait before getting a result. 

For more details for how to use `get_task`, check out [Instructions for creating your first Task for a MTurk single purpose API].

#### Example request

```
client.get_task('intent-detection', 'my-task-name')
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
|"matches" | matching intents selected by Workers   | Array of JSON Objects |
|"label" | the label representing the name of a single intent | String |

> Currently, the API only returns one match, as an array of one.

#### Example response

Example response for a successful Task:

```
{
  "taskName": "my-task-name",
    "input": {
     "text": "my son was stung by bees and I need to know if I need to go to the ER",
     "intents":[
        {"label": "Schedule an appointment", 
         "description": "Caller expresses an intent to visit with health care professional"
         "positiveExamples":  [
               {"text": "I need to make an appointment with Dr. Smith" },
               {"text": "Can the doctor see me today?"}], 
         "negativeExamples":  [
               {"text": "Do you sell burritos?" },
               {"text": "Do you accept credit cards?"}] },
        {"label": "Medical Record Request", 
         "description": "Caller expresses a desire to obtain copies of their medical history"
         "positiveExamples":  [
               {"text": "I need a copy of my kids’ immunization records" },
               {"text": "Can you fax my test results to my surgeon"}], 
         "negativeExamples":  [
               {"text": "I need to refill my prescription"},
               {"text": "Do you have my test results?"}] }]
   },
  "problemDetails": null,
  "state": "completed",
  "result": {
    "matches": [{"label": "Schedule an appointment"}] 
  }
}
```

Example response for a failed Task:

```
{
  "taskName": "my-task-name",
    "input": {
     "text": "my son was stung by bees and I need to know if I need to go to the ER",
     "intents":[
        {"label": "Schedule an appointment", 
         "description": "Caller expresses an intent to isit with health care professional"
         "positiveExamples":  [
               {"text": "I need to make an appointment with Dr. Smith" },
               {"text": "Can the doctor see me today?"}], 
         "negativeExamples":  [
               {"text": "Do you sell burritos?" },
               {"text": "Do you accept credit cards?"}] },
        {"label": "Medical Record Request", 
         "description": "Caller expresses a desire to obtain copies of their medical history"
         "positiveExamples":  [
               {"text": "I need a copy of my kids’ immunization records" },
               {"text": "Can you fax my test results to my surgeon"}], 
         "negativeExamples":  [
              {"text": "I need to refill my prescription"},
               {"text": "Do you have my test results?"}] }]
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
This Python sample calls the `intent-detection` API using [Boto3] and the [MTurk Python client]. 

A user can optionally delete a finished Task (one whose state is either “completed” or “failed”). After you delete a Task, you can reuse the Task name for a future Task.

>Note: Deleting a Task that is still being processed (i.e., whose state is “processing”) is not allowed.

For more details for how to use `delete_task`, check out [Instructions for creating your first Task for a MTurk single purpose API].

#### Example Input

```
client.delete_task('intent-detection', 'my-task-name')
```

#### Example Output

A successful request for the `delete_task` operation returns with no errors and an empty body.

---

## Get started with intent-detection

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


[Download Sample Python code]: https://gist.github.com/AmazonMTurk/b4d114c2d84d694d149e1ffdf9e6d02a#file-intent-detection-test-py

[please contact our product team]: mailto:mturk-requester-preview@amazon.com
[on GitHub]: https://github.com/awslabs/mturk-crowd-beta-client-python

