# Set up permissions to call an MTurk single purpose API
By the end of this this tutorial you will have successfully created __a new IAM user__ with the necessary permissions to call an MTurk single purpose API

### Prerequisites
Before you start, you should have correctly set up the following:
1. an MTurk Requester account
1. an AWS account
1. linked your AWS account to your MTurk Requester account
1. installed the AWS Command Line Interface (optional)
 
*You can follow [detailed Instructions to set up these prerequisites].*

### Let’s get started.
[Identity and Access Management (IAM)] is an AWS feature that enables you to securely control access to AWS services and resources. You will create an IAM User with permissions to call a single purpose API, and as a prerequisite the permissions to call the MTurk API. Note this User will not have permission to use other AWS resources in your account unless you specify permissions to do so.

## Step 1. Create a new IAM User
On the [AWS IAM website], we will create an IAM User on the [AWS IAM website], who will have to correct permissions to call any of the single purpose APIs.

Select __Users__ from the IAM service page, then select __Add User__

![Select Add User][Add IAM User]

Enter “mturk-crowd-caller” for the name and select __Programmatic Access__. Then select __Next: Permissions__.

>Note: This User name can be customized, but you’ll need to use the same custom name everywhere else we mention “mturk-crowd-caller”

![Enter User name][Set User Details]

Select __Attach existing policies directly__, then from the __Policy type Filter__ search for __MechanicalTurk__:

![Attach existing policies][Attach existing policies]

Select the checkbox for the __AmazonMechanicalTurkFullAccess__ and __AmazonMechanicalTurkCrowdFullAccess__, then click __Next: Review__.

![Select AmazonMechanicalTurkFullAccess and AmazonMechanicalTurkCrowdFullAcces ][Create User]


Next, select __Create User__.
> __Important__:
> Now, you will see your AWS Access Key ID and Secret Access Key. You will need to configure these on your computer so you can call the API. You won’t be able to see the Secret Access Key again, so don’t close this page. If you need to, you can always generate new keys [here] following [these instructions].

![Successfully created a new IAM User][Final Add User]


Next, you will store these credentials on your computer. Leave this browser window open until we’ve saved AWS Access Key ID and Secret Access Key as described below.

## Step 2. Configure an AWS Command Line Interface (CLI) profile to call MTurk (optional)
AWS CLI allows you to define a profile that groups a set up configuration values to call an AWS service. We suggest that you set up a profile for the IAM User we just created to call MTurk.

To configure your credentials, first verify the AWS CLI is installed by entering:

```
$ aws --version
```

If you do not have the [AWS Command Line Interface] installed, you can run this command:

```
$ pip install awscli --upgrade --user
```

If you get an error, use [AWS Command Line Interface (CLI) Installation] page to troubleshoot.

Next we will create a new profile for these credentials. Enter the following:

```
$ aws configure --profile mturk-crowd-caller
```

When prompted, enter you AWS Access Key ID and AWS Secret Access Key.
> If you do not still have the browser window open with your AWS Access Key IT and AWS Secrete Access Key, generate new keys for user mturk-crowd-caller with the following instructions.

Default region name should be “us-east-1”

Default output format should be “json”

```
$ aws configure --profile mturk-crowd-caller 
 
AWS Access Key ID: AKIAIOSFON7EXAMPLE  
AWS Secret Access Key: bPxRfiCYEXAMPLEKEY  
Default region name [None]: us-east-1
Default output format [None]: json
```

If you are on a Mac or Linux computer, this should create a file at:

```
~/.aws/credential
```

If you are on a Windows computer, this file should be at:

```
C:\Users\USERNAME\.aws\credentials
```

After you run the ```aws configure``` command the file should now have the following:

```python
[mturk-crowd-caller]
aws_access_key_id = AKIAIOSFODNN7EXAMPLE
aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```

*If you decide not to set up the AWS CLI, you will need to add your AWS credentials to your Python file as described in the [Boto3 Documentation].*

	
## You are now done with permission set up
Next, find the instructions to [prepay for your MTurk HITs] or jump right to [calling your single purpose API].

You may also find this list of [New single purpose APIs, Common Questions] helpful

If you have any questions or feedback, [please contact our product team] or chat with us in our [Slack channel] by joining [via this link].

[Add IAM User]: https://cdn-images-1.medium.com/max/1600/1*xxCDJ2YBmSeu5fLMp4WjmQ.png
[Set User Details]: https://cdn-images-1.medium.com/max/2000/1*NRstuzYkVH5GTFWZB_8_Vw.png
[Attach existing policies]: https://cdn-images-1.medium.com/max/1600/1*Gr-EaAtsbdhwOL748L69tQ.png
[Create User]: https://cdn-images-1.medium.com/max/1600/1*Q-NRT8GaLjEZ6gDTkR3xiw.png
[Final Add User]: https://s3-us-west-2.amazonaws.com/mturk-docs-media/Final_Add_User.png

[requester.mturk.com]: https://requester.mturk.com/
[portal.aws.amazon.com]: https://portal.aws.amazon.com/gp/aws/developer/registration/index.html?nc2=h_ct
[requester.mturk.com/developer]: http://requester.mturk.com/developer
[AWS Command Line Interface]: http://docs.aws.amazon.com/cli/latest/userguide/installing.html
[detailed Instructions to set up these prerequisites]: step_0_setup_accounts.md
[Identity and Access Management (IAM)]: https://aws.amazon.com/iam/
[AWS IAM website]: https://console.aws.amazon.com/iam/home?region=us-east-1#/policies
[AWS IAM website]: https://console.aws.amazon.com/iam/home?region=us-east-1#/users
[here]: https://console.aws.amazon.com/iam/home?region=us-east-1#/users/mturk-crowd-caller?section=security_credentials
[these instructions]: http://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey
[AWS Command Line Interface (CLI) Installation]: https://docs.aws.amazon.com/cli/latest/userguide/installing.html
[for user mturk-crowd-caller]: https://console.aws.amazon.com/iam/home?region=us-east-1#/users/mturk-crowd-caller?section=security_credentials
[instructions]: https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey
[Boto3 Documentation]: https://boto3.readthedocs.io/en/latest/guide/configuration.html
[prepay for your MTurk HITs]: https://medium.com/@mechanicalturk/paying-for-work-from-preview-api-481ab24da26d
[calling your single purpose API]: step_2_first_task.md
[New single purpose APIs, Common Questions]: ../readme.md#what-apis-are-available
[please contact our product team]: mailto:mturk-requester-preview@amazon.com
[Slack channel]: https://amzn-mturk.slack.com/
[via this link]: https://join.slack.com/t/amzn-mturk/shared_invite/MjM4MzczOTI5MDQ3LTE1MDQ3MzU0MTItMzhlMTg2OWRhNA
