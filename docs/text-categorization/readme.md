# text-categorization API

* [Introduction](#introduction)
* [Create a Task](#create-a-task)
* [Get a Result](#get-a-result)
* [Delete Task when Task is complete](#delete-task-when-task-is-complete)
* [Get started with text-categorization](#get-started-with-text-categorization)

## Introduction
The `text-categorization` API enables you to categorize text based on a list of category you specify using Amazon Mechanical Turk (MTurk). This is useful in the field of natural language processing such as when training a model to understand the meaning of text.

The API takes as input a string of text to be analyzed, max length 400 characters, and an array of possible categories that matches the text. It returns an array of containing the categories marked as applicable to the text. When you call this API, it uses your MTurk Requester account and AWS account to automatically create HITs for Workers on MTurk to complete, and automatically processes answers from Workers and returns the result.

### Example Input:

```Python
{
  "input": {
    "text": "These are great. They do run a touch small. 
           I almost could go a half size up from my normal size." ,    
    "categories": [ 
         {"label": "Style", 
          "description":  "Customer mentions appearance or style of item." ,
          "positiveExamples": [
               {"text": "Not the color I ordered"},
               {"text": "No longer in season"} 
           ] ,
          "negativeExamples": [
               {"text": "Too large"},
               {"text": "Fabric was too thin"}
           ] 
         },
         {"label": "Fit", 
          "description":  "Customer mentions the cut or fit of item." ,
          "positiveExamples": [
               {"text": "This was too short."},
               {"text": "Doesn't work for my body type."} 
           ] ,
          "negativeExamples": [
               {"text": "I don't like the baggy look"},
               {"text": "Neckline is too low cut"}
           ] 
         }
      ]
   }
}
```

### Example Result:

```Python
{
  "result": {
    "matches": [{"label" : "Fit"}]
  }
}
```

Here’s how the input might be presented in a HIT to Workers on MTurk.

![Worker Preview of HIT](https://s3-us-west-2.amazonaws.com/mturk-sample-tasks/text-categorization-HIT-example.png "Worker Preview of HIT")

The API implements an adjudication strategy based on Worker agreement and returns results only when it reaches confidence.

---
All of the single purpose APIs follow the same REST pattern as described in our [REST API reference]. They require a function name indicating which API you want to call, and a unique Task name used to represent your input and the corresponding result. 

`text-categorization-test` is a version of the `text-categorization` API meant for testing your integration. It has the same interface as the production version, but Tasks always complete quickly with mock results.  HITs are not created on MTurk, so you don’t need to pay Workers for rewards.

> Note: This will return a random mock result when you call `get_task` with `text-categorization-test`.

The rest of this article provides instructions on using the `text-categorization` API using the MTurk Python Client.
[Download Sample Python code] for `text-categorization-test`.


## Create a Task
This Python sample calls the `text-categorization` API using [Boto3] and the [MTurk Python client]. 

Tasks are created by calling `put_task`. For more details for how to use `put_task`, check out [Instructions for creating your first Task for a MTurk single purpose API].

The body of a `put_task` request looks like this:

```
{
  "input": {
    "text": <text you want analyzed>,
    "categories": <array of user-supplied categories>
  }
}
```

##### Input structure

| Name | Description | Type | Required | 
| ---- | ----------- | ---- | :------: |
|"text" | text you want analyzed | String (between 1-400 characters) | Yes |
|"categories" | up to ten user-supplied categories, each with a label, description, and examples | Array of JSON Objects | Yes |
|"label" | the label representing the name of a single category | String (between 1-50 characters) | Yes|
|"description" | clarifying details about the category presented to Workers | String | No |
|"positiveExamples" | up to two examples that fit the single category | Array of JSON Objects | No |
|"negativeExamples" |  up to two examples that do not fit the single category | Array of JSON Objects | No |


#### Example request

```
text = 'These are great. They do run a touch small. I almost could go a half size up from my normal size.'

categories = [{ 'label': 'Style' ,
                'description' : 'Customer mentions appearance or style of item.',
                'positiveExamples':  [
		                {'text': 'Not the color I ordered' },
		                {'text': 'No longer in season'}
				],
                'negativeExamples': [
		                {'text':'Too large' },
		                {'text': 'Fabric was too thin'}
		]},
            { 'label': 'Fit', 
               'description': 'Customer mentions the cut or fit of item.',
               'positiveExamples':  [
		               {'text': 'This was too short.' },
		               {'text': 'Doesn\'t work for my body type.'}
		],
                'negativeExamples': [
				{'text':'I don\'t like the baggy look' },
				{'text': 'Neckline is too low cut'}
		]},
            { 'label': 'Quality', 
              'description': 'Customer mentions the material or quality of item.',
              'positiveExamples':  [
				{'text': 'Seams are not reinforced.' },
				{'text': 'Fabric has a low thread count.'}
		],
                'negativeExamples': [
				{'text':'Too expensive for what I recieved' },
				{'text': 'Seams ripped when I tried it on'}
		]},
            { 'label': 'Price', 
              'description': 'Customer mentiones the cost or price of item.',
              'positiveExamples':  [
				{'text': 'This is not worth the price' },
				{'text': 'The price was so great I ordered four and return the ones I don\'t like'}
		],
               'negativeExamples': [
				{'text':'I would not recommend anyone to purchase' },
				{'text': 'Craftmanship is too cheap'}
		]
	}]

put_result = crowd_client.put_task('text-categorization', 'my-task-name',                         
                             {'text': text, 'categories': categories })

```

#### Example response

```
{
  "taskName": "my-task-name",
  "input": {
    "text": "These are great. They do run a touch small. 
           I almost could go a half size up from my normal size." , 
    "categories": [ 
         {"label": "Style", 
          "description":  "Customer mentions appearance or style of item." ,
          "positiveExamples": [
               {"text": "Not the color I ordered"},
               {"text": "No longer in season"} 
           ] ,
          "negativeExamples": [
               {"text": "Too large"},
               {"text": "Fabric was too thin"}
           ] 
         },
         {"label": "Fit", 
          "description":  "Customer mentions the cut or fit of item." ,
          "positiveExamples": [
               {"text": "This was too short."},
               {"text": "Doesn't work for my body type."} 
           ] ,
          "negativeExamples": [
               {"text": "I don't like the baggy look"},
               {"text": "Neckline is too low cut"}
           ] 
         }
      ]
   },
  "problemDetails": null,
  "state": "processing",
  "result": null
}
```



## Get a Result
This Python sample calls the `text-categorization` API using [Boto3] and the [MTurk Python client]. 

After creating a Task, users can call the `get_task` to poll its current state. 

> Remember it can take some time for a Worker to complete a Task, so you might have to wait before getting a result. 

For more details for how to use `get_task`, check out [Instructions for creating your first Task for a MTurk single purpose API].

#### Example request

```
client.get_task('text-categorization', 'my-task-name')
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
|"matches" | matching categories selected by Workers   | Array of JSON Objects |
|"label" | the label representing the name of a single category  | String |


> Currently, the API only returns one match, as an array of one.


#### Example response

Example response for a successful Task:

```
{
  "taskName": "my-task-name",
  "input": {
    "text": "These are great. They do run a touch small. 
           I almost could go a half size up from my normal size." , 
    "categories": [ 
         {"label": "Style", 
          "description":  "Customer mentions appearance or style of item." ,
          "positiveExamples": [
               {"text": "Not the color I ordered"},
               {"text": "No longer in season"} 
           ] ,
          "negativeExamples": [
               {"text": "Too large"},
               {"text": "Fabric was too thin"}
           ] 
         },
         {"label": "Fit", 
          "description":  "Customer mentions the cut or fit of item." ,
          "positiveExamples": [
               {"text": "This was too short."},
               {"text": "Doesn't work for my body type."} 
           ] ,
          "negativeExamples": [
               {"text": "I don't like the baggy look"},
               {"text": "Neckline is too low cut"}
           ] 
         }
      ]
   },
  "problemDetails": null,
  "state": "completed",
  "result": {
    "matches": [{"label" : "Fit"}] 
  }
}
```

Example response for a failed Task:

```
{
  "taskName": "my-task-name",
  "input": {
    "text": "These are great. They do run a touch small. 
           I almost could go a half size up from my normal size." , 
    "categories": [ 
         {"label": "Style", 
          "description":  "Customer mentions appearance or style of item." ,
          "positiveExamples": [
               {"text": "Not the color I ordered"},
               {"text": "No longer in season"} 
           ] ,
          "negativeExamples": [
               {"text": "Too large"},
               {"text": "Fabric was too thin"}
           ] 
         },
         {"label": "Fit", 
          "description":  "Customer mentions the cut or fit of item." ,
          "positiveExamples": [
               {"text": "This was too short."},
               {"text": "Doesn't work for my body type."} 
           ] ,
          "negativeExamples": [
               {"text": "I don't like the baggy look"},
               {"text": "Neckline is too low cut"}
           ] 
         }
      ]
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
This Python sample calls the `text-categorization` API using [Boto3] and the [MTurk Python client]. 

A user can optionally delete a finished Task (one whose state is either “completed” or “failed”). After you delete a Task, you can reuse the Task name for a future Task.

>Note: Deleting a Task that is still being processed (i.e., whose state is “processing”) is not allowed.

For more details for how to use `delete_task`, check out [Instructions for creating your first Task for a MTurk single purpose API].

#### Example Input

```
client.delete_task('text-categorization', 'my-task-name')
```

#### Example Output

A successful request for the `delete_task` operation returns with no errors and an empty body.

---

## Get started with text-categorization

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


[Download Sample Python code]: https://gist.github.com/AmazonMTurk/86b8c38c0fb035bc4d506adb42e87f8d#file-text-categorization-test-py

[please contact our product team]: mailto:mturk-requester-preview@amazon.com
[on GitHub]: https://github.com/awslabs/mturk-crowd-beta-client-python

