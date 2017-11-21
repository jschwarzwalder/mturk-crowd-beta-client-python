# Instructions for creating your first Task for a MTurk single purpose API
MTurk has provided a Python client to enable you to call the single purpose APIs.  We have several single purpose APIs available, so check out the [single purpose APIs Common Questions] to see a list of all the available APIs.

By the end of this tutorial, you will have:

* validated that your accounts are correctly set up
* made your first calls to a test single purpose API
* made your first calls to a prod single purpose API with real inputs and responses

If you would like to use our new single purpose API from a different programming language than Python, prefer not to have to write code, or have any questions, please contact our product team and we’ll be happy to help you out.

### Prerequisites

* [Set up your Amazon Mechanical Turk Requester account and AWS account]
* [Set up permissions to call an MTurk single purpose API]
* Install [Python 2.7 or 3.6] and [pip]
* Install [Boto3]
* Install MTurk Python client (see below)

## Install the MTurk Python client
### For Mac/Linux
From your Mac or Linux terminal, run:

```
pip install --upgrade mturk-crowd-beta-client --ignore-installed six
```

> Note: If you get a “Permission Denied” error, you may need to prefix your command with “```sudo -H  ```” to run it with administrator permissions.


``` 
sudo -H pip install --upgrade mturk-crowd-beta-client --ignore-installed six
```

### For Windows

From the Windows Command Prompt (CMD), run:

```
pip install --upgrade mturk-crowd-beta-client --ignore-installed six
```

> Note: If you get a “Permission Denied” error you will need to right-click on the CMD program and select the “Run as administrator” option, then run the pip install command.


## Create your first Tasks
There are two versions of each API: prod and test.

The prod version asks actual Amazon Mechanical Turk (MTurk) Workers to complete HITs and requires paying for Worker reward and MTurk fees (see our pricing information here).

The test version of the API doesn’t actually publish HITs to MTurk Workers. Thus, it does not require paying for Worker reward and MTurk fees. The test APIs are useful for validating that your accounts are set up correctly and for testing programmatic integration with the API. You can pass it any input and it will return a random value as mock result.

### Step 1: Build a test
First we will call the test version of the single purpose API.

To begin, you will create a __Task__, which represents a single input and a corresponding result.

Open Python from your terminal or the Windows CMD prompt (just run the Python command), then run the file *[api-name]*-test.py to create a task.

> For this tutorial we will use the ```sentiment-analysis``` API, but all of the single purpose APIs work the same way. Check out the [single purpose APIs Common Questions] to see a list of all the available APIs.

```
$ python sentiment-analysis-test.py
```

Download [Sentiment Analysis Test Sample Code]

Alternatively, you can follow along line by line.

__Line 1 & 2:__ First we must import the Python client so we can create a session to connect to the API with AWS.

```
from mturk_crowd_beta_client import MTurkCrowdClient
from boto3.session import Session
```

__Line 8:__ The example assumes you have a local AWS profile called 'mturk-crowd-caller', but you can authenticate however you like, including by directly passing in your access key and secret key.

```
session = Session(profile_name='mturk-crowd-caller')
```

__Line 11:__ Next we need to initialize the client with our session.

```
crowd_client = MTurkCrowdClient(session)
```

__Line 15:__ Create a unique Task name

 Each Task name must be unique, since this will be used to return your Task's results. In the sample code we generate a random, unique name using the uuid library. task\_name is a string, between 1-64 characters, and can include alphanumeric characters, period (.), hyphen (-), and underscore (\_).

```
import uuid

task_name = 'my-test-task-' + uuid.uuid4().hex
```

__Line 22:__ Identify API to call

Next, you specify the name of the single purpose api to call. Since this is called function name in our [REST API Reference], in the sample code we create the variable  function\_name and set it to the name of the single purpose API we want to use.

In this example, you're first calling the ```sentiment-analysis-test``` API.
The test single purpose api does not cost anything and is useful for validating that your account is setup correctly and for testing your integration.

> Note that the test single purpose API returns random mock results, so you should use this before you create live tasks to ensure your environment is set up correctly and inputs are valid.


```
function_name = 'sentiment-analysis-test'
```

__Line 25:__ Each single purpose API has a specific Input structure that will be used to construct a Task for MTurk Workers. To make our code more readable we assigned it to a variable, but it can also be passed through inline.  An Input is often a string of text, or an image URL and label, formatted depending on the API you are using.

> Check out the [single purpose APIs Common Questions] to see a list of all the available APIs.

```
# input for sentiment-analysis-test is a string

text = 'Everything is wonderful.'
```

__Line 28:__  Create the Task by calling the ```put_task``` method of our client to send the function\_name, our task\_name, and our input.
```
crowd_client.put_task(function_name, task_name, {'text': text})
```

If successful you will get a response that includes the ```HTTP status code 201```. This means that your accounts were set up correctly and your function\_name and input are valid. 

 __Line 28-33:__  To better visualize the object returned, we have you assign the result to a variable and print out the JSON object that is returned.

```
put_result = crowd_client.put_task(function_name,
                                   task_name,
                                   {'text': text})

print('PUT response: {}'.format(
     {'status_code': put_result.status_code, 'task': put_result.json()}))

```

__Line 41-42:__ Lastly once you've created a task, you can retrieve the results with ```get_task```. You will need to provide the api you used as function\_name and your unique task\_name from when you created your Task.

```
get_result = crowd_client.get_task(function_name, task_name)

print('GET response: {}'.format(
        {'api-name': function_name, 'note': 'SAMPLE RESULTS ONLY', 
         'status_code': get_result.status_code, 'task': get_result.json()}))

```
If successful you will get a response that includes the ```HTTP status code 200```. The first time you call ```get_task``` with a test API, the ```"state"``` will be "processing", but all future calls will give you random mock results. 
 

If you didn’t get a successful response back for ```put_task``` or ```get_task```, please follow the instructions in the ```"problemDetails"``` message to resolve the issue. If it was an account setup issue, please refer back to our [account setup guide]. See [REST API Reference] for more information on HTTP status codes. If you continue to experience issues, don’t hesitate to [email us].


### Step 2: Call a prod API

If you received a successful response, you’re ready to call the prod API. Remember this operation will create HITs on MTurk for MTurk Workers, so If you didn’t prepay for HITs in your MTurk Requester account during account setup, you’ll get an ```InsufficientFunds``` error from the prod API.

Please visit [this page] to prepay in your MTurk Requester account. There’s a $1 minimum. For more details on pricing, go to [Paying for Work from a single purpose API]

To call a __prod__ function, Go ahead remove the "-test" from the ```function_name``` variable in your python file (line 18 in our [Sentiment Analysis Prod Sample Code]).

```
function_name = 'sentiment-analysis'
```

Since this is a prod Task, being annotated by MTurk Workers, it will take a few minutes for Workers to provide answers. Therefore, you’ll have to poll periodically by calling ```get_task```.

```
get_result = crowd_client.get_task('sentiment-analysis', 'YOUR TASK NAME')


print('GET response: {}'.format(
        {'api-name': function_name, 
         'status_code': get_result.status_code, 'task': get_result.json()}))

```

When you get a ```"state"``` of "completed", voilà, that means your Task was completed by MTurk Workers and you can get the corresponding result.

> If you only want to print out current state use ```print(format(get_result.json()['state']))```


For ```sentiment-analysis-test``` the result will be a JSON object that indicates ```"sentiment"``` as *positive, negative, neutral or cannot determine*

For example:

```
'result': {
    'sentiment': 'positive'
  }
```

Congrats! You’ve just successfully used MTurk using a single purpose API.

Get code samples for each use case on their specific pages listed from the [New single purpose APIs Common Questions] page.

If you have any questions or feedback, such as methods you wish our client supported, [please contact our product team], or submit a pull request adding additional functionality to our client.


[connect your Amazon Mechanical Turk Requester and AWS account]: step_1_setup_aws_user.md
[please contact our product team]: mailto:mturk-requester-preview@amazon.com
[Set up your Amazon Mechanical Turk Requester account and AWS account]: step_0_setup_accounts.md
[Set up permissions to call an MTurk single purpose API]: step_1_setup_aws_user.md
[Python 2.7 or 3.6]: https://www.python.org/downloads/
[pip]: https://pip.pypa.io/en/stable/installing/
[Boto3]: https://boto3.readthedocs.io/en/latest/guide/quickstart.html#installation
[Sentiment Analysis Test Sample Code]: https://gist.github.com/AmazonMTurk/a97a9d34dbe3246646989527f8439b90#file-sentiment-analysis-test-py
[New single purpose APIs Common Questions]: ../readme.md#what-apis-are-available
[single purpose APIs Common Questions]: ../readme.md#what-apis-are-available
[account setup guide]: step_1_setup_aws_user.md
[REST API Reference]: REST.md
[email us]: mailto:mturk-requester-preview@amazon.com
[this page]: https://requester.mturk.com/prepayments/new
[Paying for Work from a single purpose API]: https://medium.com/@mechanicalturk/paying-for-work-from-preview-api-481ab24da26d
[Sentiment Analysis Prod Sample Code]: https://gist.github.com/AmazonMTurk/7d1309070a986ef134c1d254d39577d7#file-sentiment-analysis-py

