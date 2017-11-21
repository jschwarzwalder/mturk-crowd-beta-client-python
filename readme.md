# Welcome

* [What APIs are available?](#what-apis-are-available)
* [What are these APIs?](#what-are-these-apis)
* [About Tasks](#tasks)
* [About Getting Results](#getting-results)
* [Next Steps](#next-steps)

## What APIs are available?

### Natural Language Processing

* [sentiment-analysis](docs/sentiment-analysis/readme.md#sentiment-analysis-api)
* [emotion-detection](docs/emotion-detection/readme.md#emotion-detection-api)
* [named-entity-recognition](docs/named-entity-recognition/readme.md)
* [coreference-resolution](docs/coreference-resolution/readme.md#coreference-resolution-api)
* [key-phrase-extraction](docs/key-phrase-extraction/readme.md)
* [semantic-similarity](docs/semantic-similarity/readme.md)
* [collect-utterance-for-intent](docs/collect-utterance-for-intent/readme.md#collect-utterance-for-intent-api)
* [intent-detection](docs/intent-detection/readme.md)
* [text-categorization](docs/text-categorization/readme.md)

### Computer Vision
* [bounding-box](docs/bounding-box/readme.md)
* [image-contains](docs/image-contains/readme.md)
* [image-categorization](docs/image-categorization/readme.md)
* [image-similarity](docs/image-similarity/readme.md)

## What are these APIs?

This suite of single purpose APIs simplifies how Requesters use Amazon Mechanical Turk (MTurk) by enabling you to automate most of the mechanics behind Human Intelligence Task (HIT) design, creation and approval.  

Each single purpose API was designed and tested to solve a specific use case, such as sentiment analysis of text, focused on the fields of natural language processing and computer vision.  MTurk has applied marketplace learnings from Requesters who have previously built solutions for these use cases. 

With these single purpose APIs, you can automate the creation and approval of HITs with a set of defaults, including the reward amount and HIT layout.  Calling an API just requires you to specify the input data to be labeled, e.g. an image URL, and the labels you’re looking for, e.g. cars and roads.  When the Task is complete, you get back the set of labels.

With this simpler way of getting data labeled, you can use MTurk to obtain ground truth data more quickly and accelerate the development of your machine learning models.

### How does it work?
To call any of the single purpose APIs, you must have [an MTurk Requester account] and an [Amazon Web Services (AWS) account]. When you call a single purpose API with an input, the single purpose API automatically calls the [MTurk API] using your Requester account and AWS account to create HITs. The HITs are visible on the [MTurk Worker site] with your name as the Requester name. After Workers complete your HITs, you call the API to get the result based on Worker answers.

### Do you have APIs that support use cases other than the ones listed?
We’re evaluating other common use cases to support via API in a similar way. If you have suggestions for additional APIs [please contact our product team].

### What programming languages are supported?
These are REST style APIs, therefore they can be called from any popular programming language. We currently provide a thin [Python client] to make it easy for Python developers to get started. If there is another language you’d like us to support, [please contact our product team].

### These APIs are in preview, what does that mean?
This is a first look at these APIs. We’re still experimenting with them and will make changes to the functionality. We expect to make backwards-compatible changes, but may make breaking changes in the future. You may encounter bugs or other issues when using it during this preview period. We’re excited to hear what you think about it. Please [contact our product team with your feedback].

## Tasks

### What's a Task

These single purpose APIs all operate on a Task, which represents an input and its corresponding result.  A [Human Intelligence Task (HIT)] is an existing MTurk concept that represents the a group of questions to Workers and their corresponding answers.  A Task is an abstraction that may represent multiple HITs under the covers.

### What operations on Task are supported?
All of these single purpose APIs currently provide PUT, GET and DELETE operations to create, get and delete Tasks. If there is an operation that you wish to be added, [please contact our product team].

### Are there any limits on usage?
You may have no more than 1 million active Tasks in progress at a given time. If you need to increase this limit, [please contact our product team].

### How can I publish large batches of Tasks all at once?
If you are interested in publishing large batches of Tasks, [please contact our product team].

### How long are my Tasks available after they complete?
After 90 days, your Tasks may no longer be available when you call GET.

## Getting Results
### How quickly can I expect to get results back?
Customers can generally expect to get results back in less than a day, often within a few minutes.

### What if I’m not happy with the quality of the results?
Customer feedback for these single purpose APIs is really important and valuable. Please [contact our product team to share any feedback].

### How do I pay for this?
Learn more about pricing on [Paying for Work from single purpose API]

### Do I have to handle contacts from Workers?
When you call these APIs, the HITs are created using your Requester account. Therefore, it’s possible for Workers to contact you when they have questions or feedback about the HITs. In the HIT instructions, we encourage Workers to provide detailed information so that you’re able to respond effectively. If you think the Worker’s feedback could be related to a bug with the single purpose API, please [forward the feedback to our product team].

If you have any additional questions or feedback, [please contact our product team]. or chat with us in our [Slack channel].

## Next Steps

Follow these tutorials to get started:
1. [Set up your Amazon Mechanical Turk (MTurk) Requester account and AWS account]
1. [Set up permissions to call an MTurk single purpose API]
1. [Instructions for creating your first Task for a MTurk single purpose API]

## Additional Documentation
* [REST API reference]


[an MTurk Requester account]: https://requester.mturk.com/developer
[Amazon Web Services (AWS) account]: https://aws.amazon.com/
[MTurk API]: https://aws.amazon.com/documentation/mturk/
[MTurk Worker site]: https://www.mturk.com



[Python client]: https://github.com/awslabs/mturk-crowd-beta-client-python
[contact our product team with your feedback]: mailto:mturk-requester-preview@amazon.com
[Paying for Work from single purpose API]: https://medium.com/@mechanicalturk/paying-for-work-from-preview-api-481ab24da26d
[forward the feedback to our product team]: mailto:mturk-requester-preview@amazon.com
[please contact our product team]: mailto:mturk-requester-preview@amazon.com
[contact our product team to share any feedback]: mailto:mturk-requester-preview@amazon.com
[Slack channel]: https://amzn-mturk.slack.com/
[Human Intelligence Task (HIT)]: http://docs.aws.amazon.com/AWSMechTurk/latest/AWSMechanicalTurkRequester/Concepts_HITsArticle.html

[Set up your Amazon Mechanical Turk (MTurk) Requester account and AWS account]: docs/step_0_setup_accounts.md
[Set up permissions to call an MTurk single purpose API]: docs/step_1_setup_aws_user.md
[Instructions for creating your first Task for a MTurk single purpose API]: docs/step_2_first_task.md
[REST API reference]: docs/REST.md
