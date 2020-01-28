# Module 2: Sample debug logs in production

## Sample 1% of debug logs in production

<details>
<summary><b>Install lambda-powertools-pattern-basic</b></summary><p>

1. At the project root, run the command `npm i --save @dazn/lambda-powertools-pattern-basic`.

This package gives you a simple wrapper which applies a couple of [middy](https://github.com/middyjs/middy) middlewares for your function:

* `@dazn/lambda-powertools-middleware-sample-logging`: which supports sampling debug logs. The wrapper configures this sample logging middleware to sample debug logs for 1% of invocations.

* `@dazn/lambda-powertools-middleware-correlation-ids`: which extracts correlation IDs from the invocation event and makes them available for the logger. It also supports a special correlation ID `debug-log-enabled`, which enables sampling debug logs at the user transaction (a chain of Lambda invocations) level.

* `@dazn/lambda-powertools-middleware-log-timeout`: which emits an error message for when a function times out. Normally, when a Lambda function times out, you don't get an error message from the application, which makes debugging time out errors difficult.

Now we need to apply it to all of our functions.

</p></details>

<details>
<summary><b>Wrap function handlers</b></summary><p>

1. Modify `functions/get-index.js` to require the `@dazn/lambda-powertools-pattern-basic` module (at the top of the file)

`const wrap = require('@dazn/lambda-powertools-pattern-basic')`

And use it to wrap our handler function. Change ln28:

`module.exports.handler = async (event, context) => {`

to the following (don't forget the closing `)` at the end!)

```javascript
module.exports.handler = wrap(async (event, context) => {
  ...
})
```

2. Repeat step 1 for **all the function handlers**.

3. By default, the sampling rate is going to be 1%, which would be hard for us to see it in action. So let's adjust the sampling rate to 10%.

In the `serverless.yml`, add the following attribute to `provider.environment` (**don't forget to indent**)

`SAMPLE_DEBUG_LOG_RATE: 0.1`

After this step, your `provider` section should look like this:

```yml
provider:
  name: aws
  runtime: nodejs12.x
  stage: dev
  region: eu-west-1
  environment:
    LOG_LEVEL: ${self:custom.logLevel.${self:custom.stage}, self:custom.logLevel.default}
    SAMPLE_DEBUG_LOG_RATE: 0.2
```

4. Deploy the demo app to a new `prod` stage, where we have configured the minimum log level to be `INFO` so by default, only `Info` logs should be recorded. But we should also expect the debug messages for 20% of invocation would be sampled.

`npm run sls -- deploy -s prod`

5. After deployment finishes, refresh the page and place an order.

6. Repeat step 4 a couple of times (let's say do it ten times).

7. Go to CloudWatch Logs Insights and run the query

```
fields functionName, sLevel, @timestamp, message
| sort @timestamp desc
| limit 20
| filter ispresent(functionName)
```

against the five prod functions we have just deployed.

![](/images/mod02-001.png)

See that most of the time the debug logs don't appear, only around 1/10 invocations' debug logs show up.

![](/images/mod02-002.png)

8. To avoid unnecessary costs in your AWS account, delete the `prod` environment

`npm run sls -- remove -s prod`

</p></details>