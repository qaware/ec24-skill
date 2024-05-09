# The Engineering Camp Alexa Skill

Simple Alexa demo skill to ask for the next talk in the Pretalx schedule.

```bash
# install CLI and create new skill project
npm install -g ask-cli
ask new

# install dependencies
cd lambda
npm install
npm install --save-dev ask-sdk-local-debug

# to run and test implementation locally
ask run --region EU
ask dialog
```

Once you have everything up and running you can start with the development
of the skill. Either use CoPilot or the following snippets to implement the
actual business logic.

```javascript
// a function to get the complete EC24 schedule from Pretalx
const https = require('https');
function get(url) {
    return new Promise((resolve, reject) => {
        https.get(url, (res) => {
            let data = '';

            res.on('data', (chunk) => {
                data += chunk;
            });

            res.on('end', () => {
                resolve(data);
            });

        }).on('error', (err) => {
            reject(err);
        });
    });
}

// a function to check of the timestamp is today and in the future
function isTodayAndAfterCurrentTime(timestamp) {
    const now = new Date();
    const timestampDate = new Date(timestamp);

    return now.toISOString().slice(0,10) === timestampDate.toISOString().slice(0,10) && now.getTime() < timestampDate.getTime();
}  
```

Then, add the `handle()` callback to your main intent handler. 
Done. Deploy. Test.
```javascript
    async handle(handlerInput) {
        let speakOutput = 'I could not find any talks for today. Please try again later.';

        await get('https://ec24.qaware.de/api/events/ec24/talks/').then(
            (data) => {
                console.log(data);
                const talks = JSON.parse(data);

                for (let i = 0; i < talks.results.length; i++) {
                    const talk = talks.results[i];
                    const timestamp = talk.slot.start;
                    if (isTodayAndAfterCurrentTime(timestamp)) {
                        speakOutput = `The next talk is ${talk.title} by ${talk.speakers[0].name}`;
                        break;
                    }
                }
            }
        ).catch(console.error);

        return handlerInput.responseBuilder
            .speak(speakOutput)
            .reprompt('Anything else I can do for you?')
            .getResponse();
    }
```

## Maintainer

M.-Leander Reimer (@lreimer), <mario-leander.reimer@qaware.de>

## License

This software is provided under the GPL v3 open source license, read the `LICENSE` file for details.
