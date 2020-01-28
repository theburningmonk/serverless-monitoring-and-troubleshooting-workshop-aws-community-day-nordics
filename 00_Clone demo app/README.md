# Module 0: Clone the demo project

<details>
<summary><b>Clone and deploy the demo project</b></summary><p>

1. Fork this [repo](https://github.com/theburningmonk/serverless-monitoring-and-troubleshooting-workshop-aws-community-day-nordics-demo)

and clone it on your laptop

2. Open the repo in your IDE

3. Run `npm install`

4. Open `serverless.yml` and replace `<INSERT NAME HERE>` on ln 4 with your name, e.g. `yancui`.

For this demo, we'll simulate sending a push notification via SNS. To keep things simple, we'll deliver the notification by email instead.

In the `serverless.yml`, go to ln 5 and replace `<INSERT EMAIL HERE>` with your email.

5. Deploy the demo project

`npm run sls -- deploy`

you should see console output like the following

```
Serverless: Packaging service...
Serverless: Excluding development dependencies...
Serverless: Creating Stack...
Serverless: Checking Stack create progress...
........
Serverless: Stack create finished...
Serverless: Uploading CloudFormation file to S3...
Serverless: Uploading artifacts...
Serverless: Uploading service sls-workshop-yancui.zip file to S3 (10.54 MB)...
Serverless: Validating template...
Serverless: Updating Stack...
Serverless: Checking Stack update progress...
..................................................................................................................
Serverless: Stack update finished...
Service Information
service: sls-workshop-yancui
stage: dev
region: eu-west-1
stack: sls-workshop-yancui-dev
resources: 39
api keys:
  None
endpoints:
  GET - https://43togftd2c.execute-api.eu-west-1.amazonaws.com/dev/
  GET - https://43togftd2c.execute-api.eu-west-1.amazonaws.com/dev/restaurants
  POST - https://43togftd2c.execute-api.eu-west-1.amazonaws.com/dev/restaurants/search
  POST - https://43togftd2c.execute-api.eu-west-1.amazonaws.com/dev/orders
functions:
  get-index: sls-workshop-yancui-dev-get-index
  get-restaurants: sls-workshop-yancui-dev-get-restaurants
  search-restaurants: sls-workshop-yancui-dev-search-restaurants
  place-order: sls-workshop-yancui-dev-place-order
  notify-restaurant: sls-workshop-yancui-dev-notify-restaurant
  seed-restaurants: sls-workshop-yancui-dev-seed-restaurants
layers:
  None
Serverless: Run the "serverless" command to setup monitoring, troubleshooting and testing.
```

6. Open the first `GET` endpoint in the browser and you should see something like this.

![](/images/mod00-001.png)

7. You should also have received an email from SNS to confirm your subscription.

![](/images/mod00-002.png)

Click on the `Confirm subscription` link

![](/images/mod00-003.png)

</p></details>

<details>
<summary><b>Explore the demo project</b></summary><p>

1. Go to the landing page, and search for `cartoon`, click `Find Restaurants`

2. Click on `Fancy Eats` to place an order

![](/images/mod00-004.png)

3. Check your inbox, you should have received a notification email about the order

![](/images/mod00-005.png)

</p></details>

Congratulations! You have deployed a serverless project with:

* a `GET /` endpoint that returns the landing page HTML
* a `GET /restaurants` endpoint that returns a default list of restaurants
* a `POST /restaurants/search` endpoint to search restaurants by theme
* a `POST /orders` endpoint to place an order and publishes an `order_placed` event into a Kinesis stream
* a `notify-restaurant` restaurant function that reacts to the `order_placed` event and publishes a push notification to SNS

This is how things look from a high level:

![](/images/mod00-006.png)

But, this project is not worthy of production yet, because:

* there are no tests at all
* it's logging with `console.log`
* there's no way to disable debug logging in production
* we can't correlate log messages across multiple functions
* we don't have visibility into what's happening inside the function to debug performance issues