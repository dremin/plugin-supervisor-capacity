# Flex Supervisor Capacity Plugin

This plugin implements a *Channel Capacity* panel in the [Twilio Flex](https://www.twilio.com/flex) [Supervisor View](https://www.twilio.com/docs/flex/monitor-agent-activity). It includes code for [Twilio Functions](https://www.twilio.com/docs/runtime/functions) as well as frontend UI code in the form of a [Flex plugin](https://www.twilio.com/docs/flex/quickstart/getting-started-plugin).

![Plugin Demo](https://github.com/twilio-professional-services/plugin-supervisor-capacity/blob/media/supervisor-capacity-recording.gif)

**For the Flex UI 1.x version of this plugin, see [the master branch](https://github.com/twilio-professional-services/plugin-supervisor-capacity/tree/master).**

## Setup

### Prerequisites
Before beginning with this Flex plugin, you'll want to make sure that:
- You have a working [Twilio Flex](https://www.twilio.com/flex) account
- You have [Node.js](https://nodejs.org) as well as [`npm`](https://npmjs.com) installed
  - `npm` generally gets installed along with Node.js, but make sure you have it anyway
- You have the latest [Twilio CLI](https://www.twilio.com/docs/twilio-cli/quickstart) installed
- Your Twilio CLI is running the latest [Flex Plugin extension](https://www.twilio.com/docs/flex/developer/plugins/cli/install) and the [Serverless Plugin](https://github.com/twilio-labs/plugin-serverless) 

### Configuration
Over the course of the configuration process, you'll need several values from your Twilio account. The first three can be found right now in the Twilio Console, but the last one will require you to deploy your Twilio Functions to find (Don't worry, we'll cover that!)

- Account SID
  - Found on the [Twilio Console Homepage](https://www.twilio.com/console)
  - String starting with "AC..."
- Auth Token
  - Found on the [Twilio Console Homepage](https://www.twilio.com/console)
  - Secure string of 32 characters that we'll call "blah..." for the sake of communication
- Workspace ID
  - Found in your [TaskRouter Dashboard](https://www.twilio.com/console/taskrouter/dashboard)
  - String starting with "WS..."
- Serverless Runtime Domain
  - We'll grab this after we've deployed our Twilio Functions
  - A web domain that looks something like "foobar-xxx-dev.twilio.io"
  
We'll be entering these values into three files, some of which don't exist yet:
- public/appConfig.js
- src/Functions/.env
- src/config.js

#### public/appConfig.js
To kick things off, rename the example app configuration file to remove `.example`, then open it in your editor of choice

```bash
mv public/appConfig.example.js public/appConfig.js

vim public/appConfig.js
```

#### src/Functions/.env
Next, we'll need to configure the environment variables for the Twilio Functions. Start by renaming the environment file to remove `.example` and opening it with your editor:

```bash
mv src/Functions/.env.example src/Functions/.env

vim src/Functions/.env
```

Now, just like before, replace the temporary strings with your actual values

```
# Before
ACCOUNT_SID=accountSid
AUTH_TOKEN=authToken
TWILIO_WORKSPACE_SID=workspaceSid

# After
ACCOUNT_SID=AC...
AUTH_TOKEN=blah...
TWILIO_WORKSPACE_SID=WS...
```

#### Deploying Functions

Before we can configure the last file, we'll need to deploy our Twilio Functions and grab the Runtime Domain. To do so, we'll be using the [Twilio CLI](https://www.twilio.com/docs/twilio-cli/quickstart) and the [Serverless Plugin](https://github.com/twilio-labs/plugin-serverless) that you installed as a prerequisiste.

First off, make sure that you have authenticated according to the [Twilio CLI documentation](https://www.twilio.com/docs/twilio-cli/quickstart#login-to-your-twilio-account).

Then cd into the Functions directory and deploy them:

```bash
cd src/Functions
twilio serverless:deploy
```

Once everything gets deployed, your response should look something like this:

```bash
Deployment Details
Domain: foobar-xxx-dev.twilio.io
Service:
   plugin-supervisor-capacity-functions (ZS...)
Environment:
   dev (ZE...)
Build SID:
   ZB...
View Live Logs:
   Open the Twilio Console
Functions:
   https://foobar-xxx-dev.twilio.io/getWorkerChannels
   https://foobar-xxx-dev.twilio.io/setWorkerChannelCapacity
Assets:
```

The value we're looking for comes after `Domain:` – that's your Runtime Domain.

#### config.js

Now open `src/config.js` in your text editor. Notice the runtime domain set to a default value? Let's change that:

```javascript
# Before:
export default {
    runtimeDomain: "http://localhost:3000",
    filterChannels: false
}

# After:
export default {
    runtimeDomain: "https://foobar-xxx-dev.twilio.io",
    filterChannels: false
}

```

You may also set `filterChannels` to `true` if you would like to limit which channels can be configured, as well as their valid ranges.

#### UI configuration

If `filterChannels` was set to `true` in `config.js`, you must configure which task channels appear in the UI, and the valid range for each channel, by updating the Flex UI configuration:

```
POST https://flex-api.twilio.com/v1/Configuration
Authorization: Basic {base64-encoded Twilio Account SID : Auth Token}
Content-Type: application/json

{
  "account_sid": "Enter your Twilio Account SID here",
  "ui_attributes": {
    ... include your existing ui_attributes here ...
    "channel_capacity": {
      "voice": {
        "min": 0,
        "max": 1
      },
      "chat": {
        "min": 0,
        "max": 10
      }
    }
  }
}
```

And now your plugin is fully configured! You can now run it locally to test and customize it, or build it into a package and upload it to your Twilio Assets for hosted use.

## Local Development/Deployment

In order to develop locally, you can use the Twilio CLI to run the plugin locally. Using your commandline run the following from the root dirctory of the plugin.

```bash
twilio flex:plugins:start
```

This will automatically start up the Webpack Dev Server and open the browser for you. Your app will run on `http://localhost:3000`.

When you make changes to your code, the browser window will be automatically refreshed.

## Deploy

Once you are happy with your plugin, you have to deploy then release the plugin for it to take affect on Twilio hosted Flex.

Run the following command to start the deployment:

```bash
twilio flex:plugins:deploy --major --changelog "Notes for this version" --description "Functionality of the plugin"
```

After your deployment runs you will receive instructions for releasing your plugin from the bash prompt. You can use this or skip this step and release your plugin from the Flex plugin dashboard here https://flex.twilio.com/admin/plugins

For more details on deploying your plugin, refer to the [deploying your plugin guide](https://www.twilio.com/docs/flex/plugins#deploying-your-plugin).

Note: Common packages like `React`, `ReactDOM`, `Redux` and `ReactRedux` are not bundled with the build because they are treated as external dependencies so the plugin will depend on Flex to provide them globally.

## Disclaimer
This software is to be considered "sample code", a Type B Deliverable, and is delivered "as-is" to the user. Twilio bears no responsibility to support the use or implementation of this software.
