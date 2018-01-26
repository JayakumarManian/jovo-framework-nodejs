# [App Configuration](../) > [Server](README.md) > Webhook

For voice apps in prototyping stage, we recommend using a webhook and a tunnel service like [ngrok](#ngrok) or [bst proxy](#bst-proxy). This way, you can easily update your app without having to upload it to a server or [AWS Lambda](./aws-lambda.md) every time.

* [Webhook Configuration](#webhook-configuration)
* [Run the Webhook Locally](#run-the-webhook-locally)
  * [ngrok](#ngrok)
  * [bst proxy](#bst-proxy)
* [Deploy to a Server](#deploy-to-a-server)


## Webhook Configuration

Jovo uses the [`express`](https://expressjs.com/) framework for running a server. Here is how the part of `index.js`, which is used to run the app on a webhook, looks like:

```javascript
'use strict';

const {Webhook} = require('jovo-framework');
const {app} = require('./app/app.js');

if (isWebhook()) {
    const port = process.env.PORT || 3000;
    Webhook.listen(port, () => {
        console.log(`Example server listening on port ${port}!`);
    });
    Webhook.post('/webhook', (req, res) => {
        app.handleWebhook(req, res);
    });
}
```

You can either run your server locally, or deploy to a webhosting service.


## Run the Webhook Locally

You can use either of the following commands to run the server locally:

```sh
# Using the Jovo CLI
$ jovo run

# Alternative
$ node index.js
```

Find more information on the `jovo run` command here: [CLI](.../02_cli).

It should return this:

```sh
Local development server listening on port 3000.
```

Make sure that, with every file update, you terminate the server with `ctrl+c` and run it again.

After running the server, you can use either [ngrok](#ngrok) or [bst proxy](#bst-proxy) to create a link to a your local webhook, which can then be posted as a HTTPS endpoint to the voice platforms.

### ngrok

[Ngrok](https://ngrok.com/) is a tunneling service that makes your localhost accessible to outside APIs. This way, you can prototype locally without having to deal with servers or Lambda uploads all the time. You can download ngrok like so:

```sh
# Open a new tab in your command line tool, then:
$ npm install ngrok -g

# Point ngrok to your localhost:3000
$ ngrok http 3000
```

It should display something similar to this:

![ngrok window](https://www.jovo.tech/img/docs/building-a-voice-app/webhook-url.jpg)

Now use the `https://xyz.ngrok.io` address provided by ngrok, add `/webhook` and paste it as webhook link to the respective developer platform consoles.

Find the following sections in our beginner tutorials to learn how to do so:

* [Amazon Alexa: Add Webhook as HTTPS Endpoint](https://www.jovo.tech/blog/alexa-skill-tutorial-nodejs/#app-configuration)
* [Google Assistant: Add Webhook as Dialogflow Fulfillment](https://www.jovo.tech/blog/google-action-tutorial-nodejs/#endpoint)

### bst proxy

With the bst proxy by [Bespoken](https://bespoken.io/), you can create a link similar to ngrok, but with powerful features like logging.

You can run the proxy with the `jovo run` command:

```sh
$ jovo run --bst-proxy
```
This is what the result looks like:

![bst proxy result](https://www.jovo.tech/blog/wp-content/uploads/2017/10/terminal-bst-proxy-1.jpg)

Now, you can not only use the link as an endpoint, but also use it to access [Bespoken Analytics](.../07_integrations/analytics#bespoken) for powerful logging capabilities:

![bst proxy result](https://www.jovo.tech/blog/wp-content/uploads/2017/10/bespoken-logging.jpg)


## Deploy to a Server

When you want to deploy your code to a webserver other than AWS Lambda, you need to verify that Alexa Skill requests are actually coming from Amazon.

For this, Jovo uses a package called [alexa-verifier](https://github.com/mreinstein/alexa-verifier), which can be accessed by switching one line of the configuration in `index.js`:

```javascript
// Use this
const {Webhook} = require('jovo-framework').WebhookVerified;

// Instead of this
const {Webhook} = require('jovo-framework');
```