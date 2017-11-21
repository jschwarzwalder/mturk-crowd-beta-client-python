# Set up your Amazon Mechanical Turk (MTurk) Requester account and AWS account
Before you can use any of the MTurk APIs you need to have a Requester account and a linked AWS account. Follow these steps to set up your accounts.

## Step 1. Create an MTurk Requester account
First, If you haven’t already, register for an MTurk Requester account at [requester.mturk.com].

![requester.mturk.com][requester site]

## Step 2. Create an AWS account
Next, If you haven’t already, please register for an AWS account at [portal.aws.amazon.com].

> At the end of registration, it will ask you for a  credit card. Note that your MTurk usage will not appear in your AWS bill since you will prepay for your usage on MTurk’s website. However, if you use other services with this AWS account, you may accrue AWS charges.

![portal.aws.amazon.com][aws site]

## Step 3. Link your AWS account to your MTurk Requester
Once you have an MTurk Requester account and an AWS account, link them following the instructions at requester.mturk.com/developer.

This will allow MTurk to use the AWS account to identify you and allow access for you to use the MTurk API.

1. Log in to your Requester account on [requester.mturk.com/developer].
The first step is to create an AWS account which you’ve done.

2. Go down to Step 2, and select __Link your AWS Account__  
![Step 2 at requester.mturk.com/developer][Step 2]

3. Log into your AWS account if prompted, and confirm the Account ID. You can log in to a different AWS account from this screen if your AWS account uses a different email than your Requester account.

4. Select __Link this Account__ to continue  
![Select Link this Account to continue][Link this Account]

When your accounts are correctly linked, you will see this:

![Accounts Successfully Linked][Confirmation] 

When you scroll down to Step 2, you’ll see your AWS account number and account details.

![When your MTurk Requester Account and AWS Root User are linked.][Linked Account Display]

You can complete the rest of the steps on [requester.mturk.com/developer] at your convenience. For now we’ll proceed to set up permissions to call a single purpose API through your AWS account.


### Next you should [set up permissions to call a single purpose API] and [make your first API call].

[requester site]: https://s3-us-west-2.amazonaws.com/mturk-docs-media/1_sYyOYi6RQrOgNHb3VsEnLw%5B1%5D.png "Requester Website"
[aws site]: https://s3-us-west-2.amazonaws.com/mturk-docs-media/1_8Y0RSnfSkVEiM_RcvqyQHg%5B1%5D.png "AWS Portal"
[Step 2]: https://s3-us-west-2.amazonaws.com/mturk-docs-media/1_C7irRzBlG9EeYNIqBZRK2g%5B1%5D.png "Requester Website"
[Link this Account]: https://s3-us-west-2.amazonaws.com/mturk-docs-media/1_tH197drbZKbMTCENz5Cn7w%5B1%5D.png
[Confirmation]: https://s3-us-west-2.amazonaws.com/mturk-docs-media/1_TTXhNYOeuQnQsvEiWD2QVw%5B1%5D.png
[Linked Account Display]: https://s3-us-west-2.amazonaws.com/mturk-docs-media/1_Nt0MAd8bqsr8Eo-3uhU1pQ%5B1%5D.png "Account is now linked"

[requester.mturk.com]: https://requester.mturk.com/begin_signin
[portal.aws.amazon.com]: https://portal.aws.amazon.com/gp/aws/developer/registration/index.html?nc2=h_ct
[requester.mturk.com/developer]: https://requester.mturk.com/developer
[set up permissions to call a single purpose API]: step_1_setup_aws_user.md
[make your first API call]: step_2_first_task.md
