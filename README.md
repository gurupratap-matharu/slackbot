# A Simple Python Slackbot

Bots are a useful way to interact with chat services such as Slack. Let's build a simple Slack bot to whom you can send commands using Python using the Slack-API.

## Our Weapons
Our bot, whom we'll call "Merlin Bot", requires Python and the Slack API. To run our Python code we need:

* Python3
* pip and virtualenv to handle Python application dependencies
* Free Slack account - you need to be signed into at least one workspace where you have access to building apps.
It is also useful to have the Slack API docs handy while you're building this tutorial.

## Establishing Our Environment
Go to the terminal (or Command Prompt on Windows) and change into the directory where you want to store this project. Call the directory slackbot
Within that directory, create a new virtualenv to isolate our application dependencies from other Python projects.

```
virtualenv venv
```

Activate the virtualenv:

```
source venv/bin/activate
```

The official slackclient API helper library built by Slack can send and receive messages from a Slack channel. Install the slackclient library with the pip command:

```
pip install slackclient
```
When pip is finished you should see output like this and you'll be back at the prompt.

Upgrade pip to 18 if it prompts and see my pip list details below.

We also need to create a Slack App to recieve an API token for your bot. Use "Merlin Bot" as your App name. If you are signed into more than one workspace, pick a Development Workspace from the dropdown.

Matharu is my Workspace yours would be different. After submitting the form, keep the app configuration page open.

## Slack APIs and App Configuration

We want our Merlin Bot to appear like any other user in your team - it will participate in conversations inside channels, groups, and DMs.

In a Slack App, this is called a bot user, which we set up by choosing "Bot Users" under the "Features" section. After clicking "Add a Bot User", you should choose a display name, choose a default username, and save your choices by clicking "Add Bot User". 


The slackclient library makes it simple to use Slack's RTM API and Web API. We'll use both to implement Merlin Bot, and they each require authentication. Conveniently, the bot user we created earlier can be used to authenticate for both APIs.

Click on the "Install App" under the "Settings" section. The button on this page will install the App into our Development Workspace. Once the App is installed, it displays a bot user oauth access token for authentication as the bot user.


A common practice for Python developers is to export secret tokens as environment variables. Back in your terminal, export the Slack token with the name SLACK_BOT_TOKEN:

```
export SLACK_BOT_TOKEN='your bot user access token here'
```


Nice, now we are authorized to use the Slack RTM and Web APIs as a bot user.

The code instantiates the SlackClient client with our SLACK_BOT_TOKEN exported as an environment variable. It also declares a variable we can use to store the Slack user ID of our Merlin Bot. A few constants are also declared, and each of them will be explained as they are used in the code that follows.

The Slack client connects to the Slack RTM API. Once it's connected, it calls a Web API method (auth.test) to find Merlin Bot's user ID.

Each bot user has a user ID for each workspace the Slack App is installed within. Storing this user ID will help the program understand if someone has mentioned the bot in a message.

Next, the program enters an infinite loop, where each time the loop runs the client receives any events that arrived from Slack's RTM API. Notice that before the loop ends, the program pauses for one second so that it doesn't loop too fast and waste your CPU time.

For each event that is read, the parse_bot_commands() function determines if the event contains a command for Merlin Bot. If it does, then command will contain a value and the handle_command() function determines what to do with the command.

We've laid the groundwork for processing Slack events and calling Slack methods in the program. Next, add three new functions above the previous snippet to complete handling commands:

```
def parse_bot_commands(slack_events):"""
        Parses a list of events coming from the Slack RTM API to find bot commands.
        If a bot command is found, this function returns a tuple of command and channel.
        If its not found, then this function returns None, None.
    """for event in slack_events:
        if event["type"] == "message" and not "subtype" in event:
            user_id, message = parse_direct_mention(event["text"])
            if user_id == merlinbot_id:
                return message, event["channel"]
    return None, None

def parse_direct_mention(message_text):"""
        Finds a direct mention (a mention that is at the beginning) in message text
        and returns the user ID which was mentioned. If there is no direct mention, returns None
    """
    matches = re.search(MENTION_REGEX, message_text)
    # the first group contains the username, the second group contains the remaining messagereturn (matches.group(1), matches.group(2).strip()) if matches else (None, None)

def handle_command(command, channel):"""
        Executes bot command if the command is known
    """# Default response is help text for the user
    default_response = "Not sure what you mean. Try *{}*.".format(EXAMPLE_COMMAND)

    # Finds and executes the given command, filling in response
    response = None# This is where you start to implement more commands!if command.startswith(EXAMPLE_COMMAND):
        response = "Sure...write some more code then I can do that!"

    # Sends the response back to the channel
    slack_client.api_call(
        "chat.postMessage",
        channel=channel,
        text=response or default_response
    )
```
The parse_bot_commands() function takes events from Slack and determines if they are commands directed at Merlin Bot. There are many event types that our bot will encounter, but to find commands we only want to consider message events. Message events also have subtypes, but the commands we want to find won't have any subtype defined. The function filters out uninteresting events by checking these properties.

Now we know the event represents a message with some text, but we want to find out if Merlin Bot is being mentioned in the text. The parse_direct_mention() function will figure out of the message text starts with a mention, and then we compare that to the user ID we stored earlier for Merlin Bot. If they are the same, then we know this is a bot command, and return the command text with the channel ID.

The parse_direct_mentions() function uses a regular expression to determine if a user is being mentioned at the beginning of the message. It returns the user ID and the remaining message (and None, None if no mention was found).

The last function, handle_command() is where in the future you'll add all the interesting commands, humor, and personality for Merlin Bot. For now, it has just one example command: do. If the command starts with a known command, it will have an appropriate response. If not, a default response is used. The response is sent back to Slack by calling the chat.postMessage Web API method with the channel.

Now that all of our code is in place we can run our Merlin Bot on the command line

```
python merlinbot.py
```

In Slack, create a new channel and invite Merlin Bot or invite it to an existing channel.

## Chat bots are a great tool to automate repetitive processes. In future most sales will be done by Chat bots!

