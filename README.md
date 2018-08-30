# Demo: Transfer user between two different bots
Two simple chatbots ("Red" and "Blue") that allow the user to talk to either of them and change between them.

### Objective
Creating two separate bots, on two separate services that can interact with each other.

One bot will be called "Blue" and talk about blue stuff. The other one will be called "Red" and talk about all things red.
A user will be able to talk to a bot and ask questions. If they'd like they can request to talk to the other bot, at which point a transfer between the two bots will happen.

### Outline
Following this tutorial, you will:
* create a "blue bot" on Dialoglfow,
* create a "red bot" on Amazon Lex,
* set them up as a single bot in Zwerm, and
* use events and custom fulfillment to let the user choose which bot to talk to.

### Requirements
In order to re-build this example, you will need:
* A free [AWS Account](https://aws.amazon.com/account/). Note: The free tier will cover this example, however if you have or will exceed the limits of it, charges may apply.
* A free [Dialogflow account](https://dialogflow.com/docs/getting-started/create-account). The free [Standard Edition](https://dialogflow.com/pricing) will cover this example.
* A [Zwerm developer account](https://iam.zwerm.io/register), which is free while in public beta.  

## Build "Blue" on Dialogflow
Download the [Blue bot archive](./Blue/demo-blue.zip) we have prepared for easy import on Dialogflow.

In the Dialogflow Console, open the agent list drop-down menu in the left navigation. Select "Create New Agent" at the bottom. In the form, enter "demo-blue" as agent name, select "English -en" as Default Language (this is normally preselected) and choose a default timezone. Either create a new Google Project or choose an existing one. Click "Create" in the top right corner to continue.

After the agent has been created you'll be taken to its list of intents. Ignore these for now and select the settings symbol in the navigation on the left, then select the "Export and Import" tab. Select "Restore from Zip" and choose the archive provided. Type "Restore" in the text field and click the Restore button to confirm the import.

Last, follow [Dialogflow's guide](https://dialogflow.com/docs/reference/v2-auth-setup) to retrieve your authentication key and keep it safe. You will need it later to set up the agent with Zwerm.

## Build "Red" on Amazon Lex 
Download the [Red bot archive](./Red/Red_Bot_LEX.zip) we have prepared for easy import on Amazon Lex.

Open the AWS Console and navigate to the [Amazon Lex service home](https://us-west-2.console.aws.amazon.com/lex/home?region=us-west-2#bots:). Select "Actions" and "Import". In the pop up dialog, open the file you've just downloaded and click "Import" again. 

Note: In Amazon Lex you cannot have multiple bots or intents with the same name. If you get a conflict while importing, Lex will ask you what to do. In that case, we recommend you unpack the archive and manually edit the JSON file to avoid duplicate names before importing.

Next, click on the name of the bot in the list to open. In the top right corner, select "Build" and confirm in the dialog. This will take a bit now, but should build your bot without any errors. Once you get confirmation of the success, you can test your bot on the right-hand side of the screen. 

For now skip this step and select "Publish" next to the build button. In the dialog window, enter "demo" as alias name and click "Publish" to proceed. After a couple of minutes this will be finished.

Before you are ready for the next step, note down the following things: The name of your Lex bot ("Red"), the alias ("demo") and the region ("us-west-2"). You will need those later.
Additionally you have to create an IAM Access key and secret. Please follow [our guide for Amazon Lex](https://prefer.atlassian.net/wiki/spaces/ZWER/pages/178683958/Add+a+bot#Addabot-AmazonLex) to create them.

## Connect bots to Zwerm
Now that Red and Blue are ready to go, you can connect them to Zwerm. Open the [Zwerm IAM console](https://iam.zwerm.io/) and select "Add a bot" on your dashboard. If you haven't set up a team yet, you will have to [create one first](https://iam.zwerm.io/settings#/teams).

The "Create a new bot" form opens. Enter "RedBlue" as the bot name and the bot ID will be filled in automatically. Select "Dialogflow" as engine and set it to API Version V2. Choose and upload the authentication key file you have created in the previous section. Give it the label "Blue". Click save and you will be taken to the overview page of your bot.

Next, you will need to connect Red. In the settings list on the left, select "Engines". Choose "Amazon Lex" in the drop-down list and select "Add" to open the "Add engine" form. Give this engine the label "Red" and fill in the form with the values you noted down from the first part ("Build Red on Amazon Lex"). Select "Save" to finish connecting Red to Zwerm.

Before you move on, there are two values you should note down. For both engines, in the list click on the eye-icon to view their details. In the pop-up window, find the "Engine ID" shown at the top and write down its value or click on the little clipboard icon to copy the value.

Now it's time to see your bots in action. In the left navigation, click on "Open in app" to open the bot in the Zwerm app. Login (if necessary) and select "Testing" in the navigation. If everything is set up correctly so far, you will immediately see Blue's introduction message: "Hello. This is Blue speaking. I talk about everything that's blue."

Go ahead now and talk to Blue. Try ask it the following some of the following things, maybe even multiple times:
- What are your favorite things?
- Tell me something about what you like.
- What do you like?

For now, Blue is the only bot you can talk to. To change this, move on to the next step.

## Events and custom fulfillment to change between bot

Zwerm makes it really easy to add more complex functionality to your Dialogflow agent. The next step would be to hand over from "Blue" to "Red" when the user requests it.

Open the `demo-blue` agent in Dialogflow again, and select "Intents" in the sidebar. As you can see, you already have an intent set up for "Talk to Red". Select it in the list to edit the intent and scroll all the way down to "Responses" section at the bottom. You will now add an additional response to this intent, to instruct Zwerm to hand the user conversation over to "Red".

Click "Add Responses" and select "Custom payload". Copy the following JSON into the editor field and make sure to replace `TEAM-NAME` with the name of your team and `RED-ENGINE-ID` with the value you have written down in the previous step.
Now select "Save" at the top right of the page and wait until your agent is rebuilt. Dialogflow will notify you about this.

```json
{
  "#StaMP": true,
  "type": "event",
  "event": "zwerm.handover",
  "payload": {
    "route": "TEAM-NAME/redblue/RED-ENGINE-ID",
    "event": {
      "event": "zwerm.welcome",
      "payload": {},
      "data": {
        "query": "get started"
      }
    }
  }
}
```
What happens here is that "Blue" now returns the `zwerm.handover` event as part of its responses for this intent. This event triggers some special behaviour within Zwerm:
* The conversation route is updated to the value in `payload.route`
* A new event is send to the engine denoted by `payload.route` with the data from `payload.event`

Long story short, the user is now talking to Red and Red has received the `zwerm.welcome` event, which will trigger Red to welcome the user.

Try it out for yourself and send your Blue bot one of these messages:
* Talk to Red
* I'm bored
* Go away!


Now that you are talking to Red, what about switching back to Blue? It's a very similar process. Open the "Red" bot in AWS. Select the "TalkToBlue" intent in the left column. Scroll down to responses and click "Add Message". Change the type to "Custom Markup" and copy and paste the following code into the editor field. Don't forget to change to replace `TEAM-NAME` with the name of your team and `BLUE-ENGINE-ID` with the value from the earlier steps.

```json
{
  "$StaMP": true,
  "type": "event",
  "event": "zwerm.handover",
  "payload": {
    "route": "TEAM-NAME/redblue/BLUE-ENGINE-ID",
    "event": {
      "event": "zwerm.welcome",
      "payload": {}
    }
  }
}
```

Next scroll further down and click "Save Intent", then click "Build" in the top right corner and confirm in the modal. This will take a couple of seconds, but once done click the blue "Publish" button next to it, choose the "demo" alias and click "Publish" again. (Note: It might take a while until you receive the latest version of the Lex bot. It's best to use an anonymous user in Zwerm's testing tool.)

That's it! You can now switch between your bots as you will. When talking to "Red", try some of these messages:
* Talk to Blue
* Do something else
* Talk to the other bot
