# Module 1: Structured logging

## Apply structured logging to the project

<details>
<summary><b>Using a simple logger</b></summary><p>

I built a suite of tools to help folks build production-ready serverless applications while I was at DAZN. It's now open source: [dazn-lambda-powertools](https://github.com/getndazn/dazn-lambda-powertools).

One of the tools available is a very simple logger that supports structured logging (amongst other things).

So, first, let's install the logger for our project.

1. At the project root, run the command `npm i --save @dazn/lambda-powertools-logger` to install the logger.

Now we need to change all the places where we're using `console.log`.

2. Modify `functions/get-index.js`.

First, require the logger at the top of the file.

`const Log = require('@dazn/lambda-powertools-logger')`

Then on ln14, change:

`console.log('loading index.html...')`

to:

`Log.info('loading index.html...')`

On ln16, change:

`console.log('loaded')`

to:

`Log.info('loaded')`

I chose `info` instead of `debug` for these two messages as they represent interesting milestones as they're performed only during cold starts.

After ln29, add a debug log message to tell us how many restaurants were retrieved.

`Log.debug('retrieved restaurants...', { count: restaurants.length })`

We're using debug here as it's useful diagnostic information for debugging individual requests, but not an interesting lifecycle event (this code runs on every request). Also, notice that the number of restaurants is captured as a separate attribute. This makes it easier to find the same log message and perform filtering/arithmetic operations when we want to look for outliers - e.g. when the restaurants endpoint returned fewer restaurants than expected.

3. Modify `functions/place-order.js` and replace `console.log` with use of the logger.

First, require the logger at the top of the file.

`const Log = require('@dazn/lambda-powertools-logger')`

Replace the 2 instances of `console.log` with `Log.debug`.

On ln11:

`Log.debug('placing order...', { orderId, restaurantName})`

On ln27:

`Log.debug('published event into Kinesis', { eventType: 'order_placed '})`

Again, we extracted the interesting attributes such as `orderId`, `restaurantName` and `eventType` out of the log message so they can be easily filtered.

4. Modify `functions/notify-restaurant.js` and replace `console.log` with use of the logger.

First, require the logger at the top of the file.

`const Log = require('@dazn/lambda-powertools-logger')`

Replace the 2 instances of `console.log` with `Log.debug`.

On ln21:

```javascript
Log.debug('notified restaurant', {
  restaurantName: order.restaurantName,
  orderId: order.orderId
})
```

On ln32:

`Log.debug('published event into Kinesis', { eventType: 'restaurant_notified '})`

5. Modify `functions/get-restaurants.js` and add some debug logs.

First, require the logger at the top of the file.

`const Log = require('@dazn/lambda-powertools-logger')`

After ln14, add a debug log message:

```javascript
Log.debug('found restaurants in DynamoDB', {
  tableName,
  limit: count,
  count: resp.Items.length
})
```

6. Modify `functions/search-restaurants.js` and add some debug logs.

First, require the logger at the top of the file.

`const Log = require('@dazn/lambda-powertools-logger')`

After ln15, add a debug log message:

```javascript
Log.debug('found restaurants in DynamoDB', {
  tableName,
  limit: count,
  theme,
  count: resp.Items.length
})
```

</p></details>

<details>
<summary><b>Disable debug logging in production</b></summary><p>

This logger allows you to control the default log level via the `LOG_LEVEL` environment variable. Let's configure the `LOG_LEVEL` environment such that we'll be logging at `INFO` level in production, but logging at `DEBUG` level everywhere else.

1. Open `serverless.yml`, in the `custom` section, add a `logLevel` attribute

```yml
logLevel:
  prod: INFO
  default: DEBUG
```

After you made this change, the `custom` should look something like this (**pay attention to the indentation!**) except for the `name` and `email` attributes.

```yml
custom:
  name: <INSERT NAME HERE>
  email: <INSERT EMAIL HERE>
  stage: ${opt:stage, self:provider.stage}
  serverless-iam-roles-per-function:
    defaultInherit: true
  logLevel:
    prod: INFO
    default: DEBUG
```

2. Still in the `serverless.yml`, under `provider` section, add the following

```yml
environment:
  LOG_LEVEL: ${self:custom.logLevel.${self:custom.stage}, self:custom.logLevel.default}
```

After this change, the `provider` section should look like this (again, **pay attention to indentation!**)

```yml
provider:
  name: aws
  runtime: nodejs12.x
  stage: dev
  region: eu-west-1
  environment:
    LOG_LEVEL: ${self:custom.logLevel.${self:custom.stage}, self:custom.logLevel.default}
```

This applies the `LOG_LEVEL` environment variable (used to decide what level the logger should log at) to all the functions in the project (since it's specified under `provider`).

It references the `custom.logLevel` object (with the `self:` syntax), and also references the `custom.stage` value (this can be overriden by CLI options). So when the deployment stage is `prod`, it resolves to `self:custom.logLevel.prod` and `LOG_LEVEL` would be set to `INFO`.

The second argument, `self:custom.logLevel.default` provides the fallback if the first path is not found. If the deployment stage is `dev`, it'll see that `self:custom.logLevel.dev` doesn't exist, and therefore use the fallback `self:custom.logLevel.default` and set `LOG_LEVEL` to `DEBUG` in that case.

This is a nice trick to specify a stage-specific override, but then fall back to some default value otherwise.

</p></details>

<details>
<summary><b>Redeploy and check the logs</b></summary><p>

1. Deploy the demo project

`npm run sls -- deploy`

2. After deployment finishes, refresh the page and place an order.

3. Head over to the CloudWatch Logs Insights console [here](https://eu-west-1.console.aws.amazon.com/cloudwatch/home?region=eu-west-1#logs-insights:queryDetail=~(end~0~start~-3600~timeType~'RELATIVE~unit~'seconds~editorString~'fields*20*40timestamp*2c*20*40message*0a*7c*20sort*20*40timestamp*20desc*0a*7c*20limit*2020~isLiveTail~false~queryId~'02d5980a-b10a-460c-bd3e-2b9905842dc7~source~(~))) in the **eu-west-1** (Ireland) region.

4. Tick all 5 log groups we care about.

![](/images/mod01-001.png)

5. Click `Run Query` and you should see results like this. If no results are found, then wait a few seconds and try again. Sometimes the logs take a few moments to propagate through.

![](/images/mod01-002.png)

6. To see the corresponding function name and log level, and to filter out the `START`, `END` and `REPORT` messages. Update the query to the following:

```
fields functionName, sLevel, @timestamp, message
| sort @timestamp desc
| limit 20
| filter ispresent(functionName)
```

![](/images/mod01-003.png)

You can expand each of the log lines to get more details

![](/images/mod01-004.png)

7. Let's try another query, to see all the events we published to Kinesis

```
fields functionName, sLevel, @timestamp, message, eventType
| sort @timestamp desc
| limit 20
| filter ispresent(functionName)
| filter message like 'published event'
```

![](/images/mod01-005.png)

CloudWatch Logs Insights is a useful tool for querying your logs. However, it has several usability limits. For example, you can only query up to 20 log groups at a time, so it's not possible to search for across all of your logs when you have hundreds of functions.

Its query syntax is alos somewhat verbose, and as a result, many still prefer an ELK stack over CloudWatch Logs Insights. To see how you can export logs from CloudWatch Logs, check out [this blog post](https://theburningmonk.com/2018/07/centralised-logging-for-aws-lambda-revised-2018/) and [this SAR app](https://serverlessrepo.aws.amazon.com/applications/arn:aws:serverlessrepo:us-east-1:374852340823:applications~auto-subscribe-log-group-to-arn).

</p></details>