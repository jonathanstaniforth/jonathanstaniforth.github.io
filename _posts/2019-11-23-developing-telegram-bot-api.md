---
layout: post
title:  "Developing with the Telegram Bot API"
date:   2019-10-23 14:00:00 +0800
tags: [php]
navigation: true
---
Telegram has an API that allows external systems, called a bot, to be used inside of the Telegram app through the sending and recieving of messages, called the [Telegram Bot API](https://core.telegram.org/bots/api). Essentially, an external system holds the logic of the bot, and contains an API endpoint which Telegram calls when a user wants to interact with the bot. This interaction can occur as either direct messages in a private chat or with an **@** mention in groups. I have been experimenting with the Telegram Bot API to see what it can do, and this post provides a few notes on its use.

## Message Object
Telegram will send a [message object](https://core.telegram.org/bots/api#message) to your API's endpoint. This object can contain a lot of information, however, the core information you'll most likely need is the following:

```json
{
    "message_id": Integer,
    "chat": Chat object,
    "text": String
}
```

The **message_id** property can be used to unique identify the message within the specific chat in which it was sent, which can be used to reply specifically to the message. **chat** contains information relating to the chat in which the message was sent, and used to send a reply to that chat. The **text** property holds the actual message that was sent.

## Chat Object
As discussed above, the chat object has information that is needed to send a reply, specifically the id property. Additionally, it states the type of the chat, e.g. private or group chat, which can be used to change the behaviour of the system to suit the situation in which it's used. An example of the chat object is below:

```json
{
    "id": chat ID,
    "type": "private", "group", "supergroup", "channel"
}
```

## Message Entities
The message object can provide a list of strings that have special meanings, such as bot commands, under the [message entities](https://core.telegram.org/bots/api#messageentity) key. For example:

```json
{
    "type": "bot_command",
    "offset": 0,
    "length": 4
}
```

This can be helpful in identifying where bot commands are in the text of a message. But, you need to extract the command from the text, using the **offset** and **length** properties, to know which command was sent. Additionally, I have found that you cannot rely on the message entities list being provided, as Telegram will not always supply it in the message object. Therefore, it's best to identify the bot commands yourself, with a possible solution for use in group chats shown below:

```php
private function getCommand($text)
{
    // List of available commands
    $commands = [
        'start',
        'resume',
        'play'
    ];

    $text = str_replace('@bot_name', '', $text); // Remove the bot name from the text
    $text = trim($text); // Remove any whitespace from the beginning and end of the text
    $first_character = substr($text, 0, 1); // Extract the first character from the text

    // If the first character is a /, extract the rest of the text
    if ($first_character === '/')
    {
        $text = substr($text, 1);
    }

    $tokens = explode(' ', $text); // Create a list of words
    $available_commands = array_intersect($commands, $tokens); // Identify the commands that are found in the list of words
    
    // If no commands found, return false
    if (sizeof($available_commands) <= 0)
    {
        return false;
    }
    
    // Return the first found command
    return $available_commands[0];
}
```

## Sending a Response
To send a response message to the Telegram Bot API, create a POST request to **https://api.telegram.org/** with the following object in the body of the request:

```json
{
    "chat_id": Chat ID from chat object,
    "text": Your reply,
    "reply_to_message_id": Message ID from message object
}
```

Below is a GitHub Gist which can be used in a Laravel project to easily send responses of different types, such as texts, locations, and animations.

<script src="https://gist.github.com/jonathanstaniforth/a42160d4cefb5e4bd61fd1c56cb00845.js"></script>