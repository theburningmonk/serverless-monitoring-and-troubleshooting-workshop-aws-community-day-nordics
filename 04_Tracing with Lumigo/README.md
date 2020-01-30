# Module 4: Lumigo

## Collect invocation traces in Lumigo

<details>
<summary><b>Sign up to Lumigo and get a token</b></summary><p>

1. Head over to [lumigo.io](https://lumigo.io/) and click `Start Free Trial` on the top right corner.

2. Choose a project name, username and password and press `Get Started`.

![](/images/mod04-001.png)

3. Follow the getting started steps and you'll arrive at a screen like this, take note of the **`token`** in step 2.

![](/images/mod04-002.png)

Instead of manually instrumenting our code, we're going to use the [serverless-lumigo](https://github.com/lumigo-io/serverless-lumigo-plugin) plugin to automate it.

</p></details>

<details>
<summary><b>Auto-instrumenting with the serverless-lumigo plugin</b></summary><p>

1. Install `serverless-lumigo` as dev dependency. Go to the project root and run

`npm i --save-dev serverless-lumigo`

2. Modify `serverless.yml` and add `- serverless-lumigo` under `plugins` (**don't forget to indent!**).

Afterwards, your `plugins` section should look like this

```yml
plugins:
  - serverless-pseudo-parameters
  - serverless-iam-roles-per-function
  - serverless-lumigo
```

3. Add the following to the `custom` section of your `serverless.yml` (**remember to indent!**) and replace `<YOUR TOKEN GOES HERE>` with the token from step 2.

```yml
lumigo:
  token: <YOUR TOKEN GOES HERE>
  nodePackageManager: npm
```

4. Deploy the project

`npm run sls -- deploy`

5. Load up the landing page, and place an order. Then head back to the Lumigo dashboard.

Go to the `Transactions` view (click on the button on the left)

![](/images/mod04-003.png)

This is where you can see the individual transactions, for fetching the index page and for handling the place-order flow. You can see the logs from the relevant functions side-by-side with the architecture components.

![](/images/mod04-005.png)

![](/images/mod04-006.png)

6. Go to the `System Map` view and see an overview of the system.

![](/images/mod04-004.png)

</p></details>
