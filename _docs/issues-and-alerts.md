---
layout: docs
title: Issues and Alerts
---

**Issues** in [Seed](/) makes it very easy to monitor your Serverless app. Issues is a native, real-time Lambda monitoring and alerting service, that:

- Works out-of-the-box. No code changes or external SDKs needed.
- Sends Slack or email alerts with a complete log of the failed request.
- Autodetects all Lambda failures. Including out of memory and timeouts.
- Supports native error reporting. Use `console.error` to report any exceptions.

![Issues feed in Seed](/assets/docs/issues-and-alerts/issues-feed-in-seed.png)

Issues is currently in private beta. Head over to your app and join the waitlist. And we'll contact you once we are ready for you.

Note that native error reporting is currently supported for Node.js. If you are interested in using it with another runtime, let us know and we'll set it up for you.

### How It Works

Issues works by subscribing to the CloudWatch log groups for all your Lambda functions. It'll listen for any errors and report them to the Seed console. And it'll optionally send you a Slack or an email notification.

Traditional error reporting services make HTTP calls to 3rd party servers to report errors. This slows down your Lambda functions as it has to wait for the reporting to complete before proceeding.

Since Issues is completely native, it doesn't need any code changes or external SDKs. This means that it does not impact your application performance in anyway. It simply reads the CloudWatch logs asynchronously and reports the errors.

And you get the complete request log, plus a link to X-Ray (if configured).

![Issues request logs](/assets/docs/issues-and-alerts/issues-request-logs.png)

### Enabling Issues

Once enabled, Issues will subscribe to the CloudWatch log groups for your Lambda functions. And it'll keep it updated when you deploy your services. Finally, it'll remove these subscriptions when you remove your app or disable Issues.

Head over to your app settings > Manage Issues > and hit **Enable Issues**.

![Enable issues in app settings](/assets/docs/issues-and-alerts/enable-issues-in-app-settings.png)

This process can take a few minutes.

![Enable issues in progress](/assets/docs/issues-and-alerts/enable-issues-in-progress.png)

Note that, CloudWatch log groups can only have 1 subscriber. This means that if you are currently using another monitoring service, you'll get an error while trying to enable it. To fix this you'll need to enabled the **Use Seed to exclusively subscribe to all Lambda functions** option.

![Enable issues with exclusive flag](/assets/docs/issues-and-alerts/enable-issues-with-exclusive-flag.png)

This setting tells Seed to remove any existing subscriptions before enabling Issues.

If you are looking for a way to setup Issues while still keeping your existing subscriptions, [get in touch with us](mailto:{{ site.email }}). 

### Types of Lambda Errors

Issues will autodetect all Lambda errors including:

- Timeouts
- Out of memory errors
- Lambda failures (any unhandled exceptions)

### Native Error Reporting

While unhandled errors are automatically reported to Issues. There are a class of errors that you might want to manually report. Issues allows you to report these errors by simply using a `console.error`. You don't need to use any 3rd party SDKs or API calls.

- Handled exceptions
  
  Your code might have an exception, you might not want your Lambda function to fail. But you want to report the error. For example, when handling API requests, you might want to catch the exception and return a JSON response.

  ``` javascript
  try {
    // Your code
  } catch(error) {
    // Report this error to Issues
    console.error(error);
    console.log('All the logs in this request will appear in the Issues page');
  }
  ```

- Custom errors

  Similarly, you might want to report errors that happen outside of a try/catch block.

  ``` javascript
  // Report this error to Issues
  console.error('Check why the name in the user object is undefined');
  // Log some debug information to go along with it
  console.log({ userObject });
  console.log('All the logs in this request will appear in the Issues page');
  ```

### Custom Loggers

Issues also supports the JSON log format out of the box. For example, if you are using the [Winston Logger](https://github.com/winstonjs/winston), you can report errors by doing the following:

```  javascript
const winston = require('winston');

const logger = winston.createLogger({
  format: winston.format.combine(
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  transports: [
    new winston.transports.Console()
  ]
});

// Report an error to Issues
logger.error('Custom error from winston');
```

### Grouping Errors

Errors are grouped together using the following algorithm:

1. First if the stack trace is available, each stack trace frame is normalized and grouped by:
  - Module name
  - Normalized filename (with revision, hashes, etc. removed)
  - Normalized context line (a cleaned up version of the source code of the affected line, if provided)
2. If stack trace is not available, but exception info is, it's grouped by exception code and message.
3. If the error is not an exception, ie. `console.error('my message')`, it's group by message.

### Notifications

You can configure Issues to alert you of an error by sending you a notification (via Slack or email). Head over to your Issues settings to set this up.

![Configure notifications for issues in Issues settings](/assets/docs/issues-and-alerts/configure-notifications-for-issues-in-issues-settings.png)

Once configured, you'll receieve alerts in Slack when an error is detected.

![Issues Slack alert](/assets/docs/issues-and-alerts/issues-slack-alert.jpg)
*Issues Slack alert*

Note that, you will not be notified again by the same Issue (groups of similar errors) in 30 minutes.

### Ignoring or Resolving Issues

You might have fixed an error and would like to resolve the issue on Seed. Or you might want to ignore certain errors. You can do so from the issue page.

![Ignore or Resolve in the Issue page](/assets/docs/issues-and-alerts/ignore-or-resolve-in-the-issue-page.png)

When an issue is ignored, you will not get notified about it or about any errors that might be grouped together with it. Ignored issues are also hidden in the feed by default.

On the other hand, When an issue is marked as resolved; it will not show up in the Issues feed. If it happens again, it will re-appear in your feed. And you will get notified immediately.

![View ignored and resolved Issues](/assets/docs/issues-and-alerts/view-ignored-and-resolved-issues.png)

Note that, you can choose to view all ignored and resolved issues on your feed by hitting the switch on the top right.

### Additional IAM Permissions

To connect to your AWS account and monitor errors, Seed needs the following set of additional permissions.

``` javascript
{
  "Sid": "ManageIssues",
  "Effect": "Allow",
  "Action": [
    // Read permission: Get log groups in a deployed stack
    "cloudformation:ListStackResources",
    // Read permission: Get deployed stack information
    "cloudformation:DescribeStacks",
    // Read permission: Check existing log group subscription
    "logs:DescribeSubscriptionFilters",
    // Write permission: Add log group subscription to enable Issues
    "logs:PutSubscriptionFilter",
    // Write permission: Remove log group subscription when disabling Issues
    "logs:DeleteSubscriptionFilter",
    // Read permission: Check Lambda runtime
    "lambda:GetFunction"
  ],
  "Resource": "*"
}
```

Issues makes it really easy to monitor the errors in your Serverless app in real-time. All without having to make any changes to your code or without the use of 3rd party SDKs.