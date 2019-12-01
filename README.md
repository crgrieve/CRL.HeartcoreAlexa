# CRL.HeartcoreAlexa

This example code is for an advent calendar Alexa skill to provide a guide to integrating Umbraco Heartcore to a skill.

## Set up your skill

If you are new to Alexa skills, I suggest working through a tutorial from Amazon, they can explain better than I can :)

[Getting Started Guide](https://developer.amazon.com/en-US/alexa/alexa-skills-kit/start)

Once you are ready to start, go to the [Alexa Skills Kit](https://developer.amazon.com/alexa/console/ask) and select "Create SKill". For this example, select "Custom" and "Alexa-hosted(Node.js)".

![](images/skillsetup.PNG)

Set up an innvocation name, this is how you ask Alexa to open your skill. For example: 'Heartcore Advent'

You can then set up an Intent. This is basically how users can invoke a function in your code. So in this case, I want users to be able to open a door in the calendar. So I have an intent that allow users to say various statements that call this code.

![](images/intentSetup.PNG)

## The Umbraco Setup
I have set up 2 doc types:

* AdventCalendar
* AdventCalendarDay - child nodes of AdventCalendar doctype, will represent each day.
** Day -  Numeric field representing the day in December
** Message - The message spoken by Alexa when this door is opened.

![](images/umbracoContent.PNG)

## The code

When you follow the set up instructions above, Amazon make life easy and create the skill template for you! Go to the 'Code' tab in the developer console. Have a wee look at this code to understand how skills code works. Remember, intents are how users interact with our code.

Ok, but how do we integrate with our Umbraco Setup? First of all, we need to set up the skill to use Heartcore. Add this to the top of your file, updatign with your Heartcore project alias. Note: if you have protected your content API, you will need to provide and API key.

```javascript
const {Client} =require('@umbraco/headless-client');
const Alexa = require('ask-sdk-core');

const client = new Client({
  projectAlias: 'your-project-alias',
  language: 'en-GB'
})
```


I have amended the `LaunchRequestHandler` and added `CalendarDayIntentHandler`. Note, we need to make the handlers 'handle' `async` as our Heartcore code is called with `await`.

```javascript
const LaunchRequestHandler = {
    canHandle(handlerInput) {
        return Alexa.getRequestType(handlerInput.requestEnvelope) === 'LaunchRequest';
    },
    async handle(handlerInput) {
        const rootContent = await client.delivery.content.byContentType('AdventCalendar');
        const contentItem = rootContent.items[0];
        const speakOutput = contentItem.name; 
        return handlerInput.responseBuilder
            .speak(speakOutput)
            .reprompt(speakOutput)
            .getResponse();
    }
};
```

```javascript
const CalendarDayIntentHandler = {
    canHandle(handlerInput) {
        return Alexa.getRequestType(handlerInput.requestEnvelope) === 'IntentRequest'
            && Alexa.getIntentName(handlerInput.requestEnvelope) === 'CalendarDayIntent';
    },
    async handle(handlerInput) {
        const calendarNodes = await client.delivery.content.byContentType('AdventCalendar');
        const contentItem = calendarNodes.items[0];
        const calendarChildren = await client.delivery.content.children(contentItem._id);
        const today =  new Date();
        const date1 = today.getDate();
        const todayNode = calendarChildren.items[date1 -1];
        const speakOutput = todayNode.message;
        return handlerInput.responseBuilder
            .speak(speakOutput)
            .getResponse();
    }
};
```


Note: you will need to make sure your new intent handler is in the `exports.handler` function. Scroll to the bottom of your file.

I the 'Code' tab you will see there is a `package.json` file. Make sure to add the Umbraco Heartcode package reference here. For example, at time of writing, these were the verisons in mine:

```json
  "dependencies": {
    "ask-sdk-core": "^2.6.0",
    "ask-sdk-model": "^1.18.0",
    "aws-sdk": "^2.326.0",
    "@umbraco/headless-client": "^0.2.0"
  }
```



## Testing your skill

When you are done you can select 'Deploy' and go on to 'Test' tab where there is an interface to type or speak to Alexa to test your skill. 


![](images/testingSkill.PNG)