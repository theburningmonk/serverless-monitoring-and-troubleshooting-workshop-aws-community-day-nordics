# Module 3: X-Ray

## Collect invocation traces in X-Ray

<details>
<summary><b>Integrating with X-Ray</b></summary><p>

1. In the `serverless.yml` under the `provider` section, add the following:

```yml
tracing:
  apiGateway: true
  lambda: true
```

2. Add the following back to the `provider` section:

```yml
iamRoleStatements:
  - Effect: Allow
    Action:
      - xray:PutTraceSegments
      - xray:PutTelemetryRecords
    Resource: "*"
```

This enables X-Ray tracing for all the functions in this project. However, we still need to give each function the IAM permission for `xray:PutTraceSegments` and `xray:PutTelemetryRecords`.

Notice that in the `custom` section, we have the following:

```yml
serverless-iam-roles-per-function:
  defaultInherit: true
```

This tells the `serverless-iam-roles-per-function` plugin that all the functions' IAM roles should inherit from the permissions specified in `provider.iamRoleStatements`, so now every function would have the necessary permissions to talk to X-Ray.

The `provider` section should now look like this

```yml
provider:
  name: aws
  runtime: nodejs12.x
  stage: dev
  region: eu-west-1
  environment:
    LOG_LEVEL: ${self:custom.logLevel.${self:custom.stage}, self:custom.logLevel.default}
    SAMPLE_DEBUG_LOG_RATE: 0.1
  tracing:
    apiGateway: true
    lambda: true
  iamRoleStatements:
    - Effect: Allow
      Action:
        - xray:PutTraceSegments
        - xray:PutTelemetryRecords
      Resource: "*"
```

3. Deploy the project

`npm run sls -- deploy`

4. Load up the landing page, and place an order. Then head to the X-Ray console and see what you get.

![](/images/mod03-001.png)

![](/images/mod03-002.png)

![](/images/mod03-003.png)

</p></details>

<details>
<summary><b>Instrumenting AWSSDK</b></summary><p>

At the moment we're not getting a lot of value out of X-Ray. We can get much more information about what's happening in our code if we instrument the various steps.

To begin with, we can instrument the AWS SDK so we track how long calls to DynamoDB and SNS takes in the traces.

1. Install `aws-xray-sdk-core` as dependency. Go to the project root and run

`npm i --save aws-xray-sdk-core`

2. Modify `functions/get-restaurants.js` and replace `const AWS = require('aws-sdk')` on ln3 with the following

```javascript
const AWSXRay = require('aws-xray-sdk-core')
const AWS = AWSXRay.captureAWS(require('aws-sdk'))
```

3. Repeat step 2 for

* `functions/place-order.js`

* `functions/search-restaurants.js`

* `functions/notify-restaurant.js`

4. Deploy the project

`npm run sls -- deploy -s dev -r eu-west-1`

5. Load up the landing page, and place an order. Then head to the X-Ray console and see what you get now.

![](/images/mod03-004.png)

![](/images/mod03-005.png)

![](/images/mod03-006.png)

![](/images/mod03-007.png)

</p></details>

<details>
<summary><b>Instrumenting HTTP calls</b></summary><p>

We can get a lot value if we could see the traces for `get-index` function and the corresponding trace for the `get-restaurants` function in one screen.

![](/images/mod03-008.png)

Then it's proper distributed tracing! It's not very helpful if you're restricted to only what happens inside one function.

Fortunately, you can instrument the built-in `https` module with the X-Ray SDK.

1. Modify `functions/get-index.js` and just before ln5

`const http = require('superagent-promise')(require('superagent'), Promise)`

insert the following

```javascript
const AWSXRay = require('aws-xray-sdk-core')
AWSXRay.captureHTTPsGlobal(require('https'))
```

After this change, you should have something like this:

```javascript
const wrap = require('@dazn/lambda-powertools-pattern-basic')
const Log = require('@dazn/lambda-powertools-logger')
const fs = require("fs")
const Mustache = require('mustache')
const AWSXRay = require('aws-xray-sdk-core')
AWSXRay.captureHTTPsGlobal(require('https'))
const http = require('superagent-promise')(require('superagent'), Promise)

// the rest of the file...
```

2. Deploy the project

`npm run sls -- deploy -s dev -r eu-west-1`

3. Load up the landing page, and place an order. Then head to the X-Ray console and now you can see the traces for `get-index` and `get-restaurants` function in one place.

</p></details>
