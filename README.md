## AWS Serverless Twitter Event Source

This serverless app turns a twitter search query into an AWS Lambda event source by invoking a given lambda function to process tweets found by search. It works by periodically polling the freely available public [Twitter Standard Search API](https://developer.twitter.com/en/docs/tweets/search/overview/standard) and invoking a lambda function you provide to process tweets found.

## Architecture

![App Architecture](https://github.com/awslabs/aws-serverless-twitter-event-source/raw/master/images/app-architecture.png)

1. The TwitterSearchPoller lambda function is periodically triggered by a CloudWatch Events Rule.
1. In stream mode, a DynamoDB table is used to keep track of a checkpoint, which is the latest tweet timestamp found by past searches.
1. The poller function calls the Twitter Standard Search API and searches for tweets using the search text provided as an app parameter.
1. The TweetProcessor lambda function (provided by the app user) is invoked with any new tweets that were found after the checkpoint timestamp.
1. If stream mode is not enabled, the TweetProcessor lambda function will be invoked with all search results found, regardless of whether they had been seen before.
    1.  Note, the TweetProcessor function is invoked asynchronously (Event invocation type). The app does not confirm that the lambda was able to successfully process the tweets. If you're concerned about tweets being lost due to failures in your lambda function, you should configure a DLQ on your lambda function so failed messages will end up on the DLQ automatically. See the [AWS Lambda DLQ documentation](https://docs.aws.amazon.com/lambda/latest/dg/dlq.html) for more information.

## Installation Steps

1. [Create an AWS account](https://portal.aws.amazon.com/gp/aws/developer/registration/index.html) if you do not already have one and login
1. Go to the app's page on the [Serverless Application Repository]() and click "Deploy"
1. Provide the required app parameters (see below for steps to create Twitter API parameters, e.g., Consumer Key)

### Twitter API Key Parameters

The app requires the following Twitter API parameters: Consumer Key (API Key), Consumer Secret (API Secret), Access Token, and Access Token Secret. The following steps walk you through registering the app with your Twitter account to create these values.

1. Create a [Twitter](https://twitter.com/) account if you do not already have one
1. Register a new application with your Twitter account:
    1. Go to [http://twitter.com/oauth_clients/new](http://twitter.com/oauth_clients/new)
    1. Click "Create New App"
    1. Under Name, enter something descriptive, e.g., `aws-serverless-twitter-es`
    1. Enter a description
    1. Under Website, you can enter `https://github.com/awslabs/aws-serverless-twitter-event-source`
    1. Leave Callback URL blank
    1. Read and agree to the Twitter Developer Agreement
    1. Click "Create your Twitter application"
1. (Optional, but recommended) Restrict the application permissions to read only
    1. From the detail page of your Twitter application, click the "Permissions" tab
    1. Under the "Access" section, make sure "Read only" is selected and click the "Update Settings" button
1. Generate an access token:
    1. From the detail page of your Twitter application, click the "Keys and Access Tokens" tab
    1. Scroll down to the Access Token section and click "Create my access token"
1. Copy the API key information from your Twitter account into the parameters for the serverless application to finish the deployment

### Other Parameters

In addition to the Twitter API key parameters, the app also requires the following additional parameters:

1. SearchText (required) - This is the **non-URL-encoded** search text the app will use when polling the Twitter Standard Search API. See the [Search Tweets](https://developer.twitter.com/en/docs/tweets/search/guides/standard-operators) help page to understand the available features. The [Twitter Search page](https://twitter.com/search) is a good place to manually test different searches, although note the standard search API generally returns different results, because it only indexes a sampling of tweets.
1. TweetProcessorFunctionName (required) - This is the name (not ARN) of the lambda function that will process tweets generated by the app.
1. PollingFrequencyInMinutes (optional) - The frequency at which the lambda will poll the Twitter Search API (in minutes). Default: 1.
1. BatchSize (optional) - The max number of tweets that will be sent to the TweetProcessor lambda function in a single invocation. Default: 15.
1. StreamModeEnabled (optional) - If true, the app will save the latest timestamp of the previous tweets found and only invoke the tweet processor function for newer tweets. If false, the app will be stateless and invoke the tweet processor function with all tweets found in each polling cycle. Default: true.

## License Summary

This sample code is made available under a modified MIT license. See the LICENSE file.
