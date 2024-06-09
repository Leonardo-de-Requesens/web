---
title: "Android Notifications for When a Process Ends in Linux"
date: 2024-06-09T12:00:34-04:00
draft: False
tags:
    - c
    - terminal
---
I was helping my wife to scrape some website with a really noob code that I made
and since the page was loading slowly the script did break several times and I had
to be waiting this to happen since I didn't have a way to check the process from
other room of my house.

This issue gave me the idea to generate an android app that could show me notifications
about some process that I was running on my machine but that it was long enough that
I really didn't wanted to wait it watching my pc.

## Receiving notifications (idea 1)
An app written in something that could show notifications of the exit code of my code.

First issue: I don't like mobile developing.

Second issue: in my last job I had to do something kind of similar and the result
of this was having to send the information to google, to use something that I don't
remember too well but had to do with Firebase ([this](https://firebase.google.com/docs/cloud-messaging/))
and then have the application running in a way that it could get the notifications
and show them... So it was too mucho for a simple "I want to know when the thing ended".

## Receiving notifications (idea 2)
Telegram bot. I remember an old coworker saying something about coding a bot for
telegram to help pet owners when their pets get lost, so it should be easier, I
don't need to code another app and I don't need a Firebase account.

### Creating a telegram bot
Is really simple, you search for @BotFather and follow the instructions to generate
a new bot, after that store the token and try it. You could generate a new chat with
the bot or a new group. After having this you need to send a **POST** request to
`https://api.telegram.org/bot<token>/getUpdates` , yes the "bot" string has to be
before the token. You are going to get something like this:
```json
{"ok":true,"result":[]}
```
If that is the case you need to send it a message, after that:
```json
{"ok":true,"result":[{"update_id":<update_id>,
"message":{"message_id":<msg_id>,"from":{"id":<user_id>,"is_bot":false,"first_name":<user_name>,"language_code":"en"},"chat":{"id":<chat_id>,"first_name":<name>,"type":"private"},"date":<date_ts>,"text":"Hello"}}]}
```
That "Hello" was the message that I send to the bot, so we have the `chat_id` and
we can do a **GET** request to `https://api.telegram.org/bot<token>/sendMessage?chat_id=<chat_id>&text=Hello%20World` giving you:
```json
{"ok":true,"result":{"message_id":<message_id>,"from":{"id":<id>,"is_bot":true,"first_name":"<bot_name>","username":"<bot_username>"},"chat":{"id":<chat_id>,"first_name":"<you>","type":"private"},"date":<date_ts>,"text":"Hello World"}}
```

We have the bot ready
## Sending the information
I want to do something like this: `./run | notification`, this means I have to read
stdout and stderror so I could send helpful messages, for the moment I will send
the full message like in the terminal, I could beautify it but not for now.

I was able to generate a simple code using libcurl to send GET request to telegram
and have the simple notification, no problem with that.

### The problems
First: the code is reading 1000 chars in a buffer and re allocate memory when needed,
this, I didn't tested it, it could explote tomorrow, also for the moment I'm not
sending partial messages because I think it will break 1/100 times so not a big issue.

Second: while I was finishing the code I used `sprintf` to format the final url
string and started to have errors:
```
malloc(): unaligned tcache chunk detected
```
I didn't understand why this was happening so I run it with `valgrind` and started
working, I received notifications but the code still crashed. This took me a while
but reading the [gnu documentation](https://www.gnu.org/software/libc/manual/html_node/Formatted-Output-Functions.html) of this function I noticed a little note

>**Warning:** The `sprintf` function can be **dangerous** because it can potentially
output more characters than can fit in the allocation size of the string s.
Remember that the field width given in a conversion specification is only a _minimum_ value.
>To avoid this problem, you can use `snprintf` or `asprintf`, described below.

This was exactly was happening I counted the amount of chars in the url really badly
and was writing to memory that I didn't ask for, and when I asked curl to generate
the request It couldn't since the memory was in a bad state. So I changed it to
`snprintf` to verify and fix.

Regarding why `valgrind` magically fixed part of the issue _I think_ it's because
`valgrind` adds code when running your code, when you check for errors you will see
that the stack trace has calls to `vg_` libraries like `vg_replace_malloc`.

---
Really nice project to understand better how Linux uses pipes and have a reason
to read documentation. I also read `wc` and `sort` code from gnu coreutils to
understand better how good code was written.

You can get the code here: [github](https://github.com/lderequesensS/notifymeplz/)
