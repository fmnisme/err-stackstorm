Activate development has moved to https://github.com/nzlosh/err-stackstorm
---

# err-stackstorm
A plugin to run StackStorm actions, bringing StackStorm's chatops to Errbot.

## Table of Contents
1. [Installation](#Installation)
1. [Requirements](#Requirements)
1. [Supported Chat Backends](#SupportedChatBackends)
1. [Configuration](#Configuration)
1. [Setup Action-Aliases](#ActionAliases)
1. [Webhook](#Webhook)
1. [Server-Side Events](#ServerSideEvents)
1. [Chatops Pack](#ChatopsPack)
1. [Troubleshooting](#Troubleshooting)

## Installation <a name="Installation"></a>
Installation of the err-stackstorm plugin is performed from within a running Errbot instance.  Ensure Errbot is up and running before attempting to install the plugin.  See the Errbot installation documentation here https://github.com/Errbotio/Errbot for instructions on how to setup Errbot on your chat back-end.  These instructions assume a running instance of StackStorm is already in place.  See the official [StackStorm documentation](https://docs.stackstorm.com/install/index.html) for details.

 1. Install Errbot on the target system using standard package manager or Errbot installation method.
 1. Configure Errbot, see the [Configuration](#Configuration) section for help.
 1. Enable Errbot's internal web server, see the [Webhook](#Webhook) section for help.
 1. Install Chatops pack on StackStorm, see the [Chatops Pack](#ChatopsPack) section for help.
 1. Connect to your chat back-end and starting interacting with your StackStorm/Errbot instance.

The below command will install the plugin.
```
!repos install https://github.com/fmnisme/err-stackstorm.git
```

## Requirements <a name="Requirements"></a>
The plugin has been developed and tested against the below software.  For optimal operation it is recommended to use the following versions:

plugin tag (version) | Python | Errbot | StackStorm client
--- | --- | --- | ---
1.4 | 3.4 | 5.1.2 | 2.5
1.3 | 3.4 | 5.1.2 | 2.5
1.2 | 3.4 | 5.0   | 2.2
1.1 | 3.4 | 4.3   | 2.2
1.0 | 2.7 | 3.x   | 2.1

## Supported Chat Back-ends <a name="SupportedChatBackends"></a>
Errbot provides official support for a few of major chat back-ends and many more chat back-ends are available through unofficial plugins.

Back end | Mode value | Support type
--- | --- | ---
Hipchat | `hipchat` | Integrated
IRC | `irc` | Integrated
Slack | `slack` | Integrated
Telegram Messenger | `telegram` | Integrated
Text | `text` | Integrated
XMPP | `xmpp` | Integrated
[Skype](https://www.skype.com/) | `skype` | [Plugin](https://github.com/errbotio/errbot-backend-skype)
[Mattermost](https://about.mattermost.com/) | `mattermost` | [Plugin](https://github.com/Vaelor/errbot-mattermost-backend)
[Rocket Chat](https://rocket.chat/) | `aoikrocketchaterrbot` | [Plugin](https://github.com/AoiKuiyuyou/AoikRocketChatErrbot)
[Glip](https://glip.com/) | `Glip` | [Plugin](https://github.com/ringcentral/ringcentral-glip-errbot)
[Gitter](gitter.im/) | `gitter` | [Plugin](https://github.com/errbotio/err-backend-gitter)
[VK](https://vk.com/) | `VK` | [Plugin](https://github.com/Ax3Effect/errbot-vk)
[Discord](https://www.discordapp.com/) | `discord` | [Plugin](https://github.com/gbin/err-backend-discord)
[Cisco Spark](https://www.ciscospark.com/) | `CiscoSpark` | [Plugin](https://github.com/marksull/err-backend-cisco-spark)
[TOX](https://tox.im/) | `tox` | [Plugin](https://github.com/errbotio/err-backend-tox)
[CampFire](https://campfirenow.com/) | `campfire` | [Plugin](https://github.com/errbotio/err-backend-campfire)

Back-end support will provide a minimum set of back-end chat functionality to the err-stackstorm plugin like `connect` to and `authenticate` with chat back-end, `identify` users/rooms and `send_message` to users/rooms.  Advanced formatting may not be available on all back-ends since adaptor code is required in the err-stackstorm plugin to translate ActionAlias `extra` parameter on a per back-end basis.

Currently supported extra back-ends
* Slack


## Configuration <a name="Configuration"></a>
Edit the `config.py` configuration file which is used to describe how the plugin will communicate with StackStorm's API and authentication end points.
If you followed the Errbot setup documentation this file will have been created by downloading a template from the Errbot GitHub site.   If this file has not already been created, please create it following the instructions at https://github.com/Errbotio/Errbot

```
STACKSTORM = {
    'auth_url': 'https://stackstorm.example.com/auth/v1',
    'api_url': 'https://stackstorm.example.com/api/v1',
    'stream_url': 'https://stackstorm.example.com/stream/v1',

    'verify_cert': True,
    'api_auth': {
        'user': {
            'name': 'my_username',
            'password': "my_password",
        },
        'token': "<User token>",
        'key': '<API Key>'
    },
    'timer_update': 900, #  Unit: second.  Interval for Errbot to refresh to list of available action aliases.
}
```

Option | Description
--- | ---
`auth_url` | StackStorm's authentication url end point.  Used to authenticate credentials against StackStorm.
`api_url` | StackStorm's API url end point.  Used to execute action aliases received from the chat back-end.
`stream_url` | StackStorm's Stream url end point.  Used to received chatops notifications.
`verify_cert` | Default is *True*.  Verify the SSL certificate is valid when using https end points.  Applies to all end points.
`api_auth.user.name` | Errbot username to authenticate with StackStorm.
`api_auth.user.password` | Errbot password to authenticate with StackStorm.
`api_auth.token` | Errbot user token to authenticate with StackStorm.  Used instead of a username/password pair.
`api_auth.key` | Errbot API key to authenticate with StackStorm.  Used instead of a username/password pair or user token.
`timer_update` | Unit: seconds.  Default is *60*. Interval for Errbot to refresh to list of available action aliases.  (deprecated)


### Authentication <a name="Authentication"></a>
Authentication is possible with username/password, User Token or API Key.  In the case of a username and password, the plugin is requests a new User Token after it expires.  In the case of a User Token or API Key, once it expires, the Errbot plugin will no longer have access to the st2 API.

The Errbot plugin must have valid credentials to use StackStorm's API.  The credentials may be;

 - username/password
 - user token
 - api key

See https://docs.stackstorm.com/authentication.html for more details.

#### Username/Password
Using a username and password will allow Errbot to renew the user token when it expires.  If a _User Token_ is supplied, it will be used in preference to username/password authentication until the token expires.

#### User Token
To avoid using the username/password pair in a configuration file, it's possible to supply a pre-generated _User Token_ as generated by StackStorm.  Note when the token expires, a new one must be generated and updated in `config.py` which in turn requires Errbot to be restarted.
This method is the least ideal for production environments.

#### API Key
_API Key_ support has been included since StackStorm v2.0.  When an _API Key_ is provided, it is used in preference to a _User Token_ or _username/password_ pair.  It is considered a mistake to supply a token or username/password pair when using the API Key.

## How to expose action-aliases as plugin commands <a name="ActionAliases"></a>
 1. Connect Errbot to your chat environment.
 1. Write an [action alias](https://docs.stackstorm.com/chatops/aliases.html) in StackStorm.
 1. Errbot will automatically refresh its action alias list.
 1. Type `!st2help` in your chat program to list available StackStorm commands.
 1. Type the desired command in your chat program, as shown in the help.

## Send messages from StackStorm to Errbot using Errbot's native webhook support <a name="Webhook"></a>

Errbot has a built in web server which is configured and enabled through the bots admin chat interface.  The StackStorm plugin is written to listen for StackStorm's chatops messages and delivers them to the attached chat back-end.

To configure Errbot's web server plugin, the command below can be sent to Errbot:
```
!plugin config Webserver {'HOST': '0.0.0.0', 'PORT': 3141,
'SSL': {'enabled': False, 'host': '0.0.0.0', 'port': 3142, 'certificate': '', 'key': ''}}
```

**NOTE:** _The variables must be adjusted to match the operating environment in which Errbot is running.  See Errbot documentation for further configuration information._

The configuration above is only applied for the current runtime and will not
persist after the errbot process being restarted. Making the configuration
change permanent is as simple as installing a special plugin:
```
!repos install https://github.com/tkit/errbot-plugin-webserverconfiguration
```
The configuration command from above is not required prior to installing this
plugin.

In production environments it may be desirable to place a reverse-proxy like nginx in front of errbot.

## Send notifications to Errbot from StackStorm using Server-Side Events (SSE) <a name="ServerSideEvents"></a>

As of StackStorm 1.4. server-sent events (SSE) were added which allowed chatops messages to be
streamed from StackStorm to a connected listener (err-stackstorm in our case).  The StackStorm
stream url must be supplied in the configuration so err-stackstorm knows where to establish the
http connection.  The SSE configuration is complementary to the webhook method and both must be
enabled for full chatops support between StackStorm and Errbot.

## StackStorm Chatops pack configuration. <a name="ChatopsPack"></a>

StackStorm's [chatops pack](https://github.com/StackStorm/st2/tree/master/contrib/chatops) is required
to be installed and a notify rule file added to the pack.

The notify rule must be placed in `/<stackstorm installation>/packs/chatops/rules`.  The rule file
[notify_errbot.yaml](https://raw.githubusercontent.com/fmnisme/err-stackstorm/master/contrib/stackstorm-chatops/rules/notify_errbot.yaml) can be found
in this repository under

Edit the `chatops/actions/post_message.yaml` file to use the errbot route as it's default value.
```
  route:
    default: "errbot"
```

## Troubleshooting <a name="Troubleshooting"></a>

### Is the Errbot process running?
Check an instance of Errbot is running on the host
```
# ps faux | grep errbo[t]
root     158707  0.1  0.0 2922228 59640 pts/21  Sl+  Aug14   2:29  |   \_ /opt/errbot/bin/python3 /opt/errbot/bin/errbot -c /data/errbot/etc/config.py
```

### Is the Errbot webhook listening?
Check Errbot's internal web server is listening on the correct interface.
```
# ss -tlpn | grep 158707
LISTEN     0      128                       *:8888                     *:*      users:(("errbot",158707,21))
```
OR
```
# netstat -tlpn | grep 158707
tcp        0      0 0.0.0.0:8888            0.0.0.0:*               LISTEN      158707/python3
```

### Is the Errbot machine able to communicate with the StackStorm end points?
From the errbot machine perform a curl to the StackStorm endpoint:

```
curl http://<stackstorm_host>/api/v1/rules
```

### Are the Errbot authentication credentials for StackStorm correct?
To test if the username/password pair, user token or api key supplied in the configuration is valid.
In the examples below, the username for the bot is `errbot`.:

#### username/password pair
A successful username / password authentication is shown below:
```
$ st2 auth errbot
Password:
+----------+----------------------------------+
| Property | Value                            |
+----------+----------------------------------+
| user     | errbot                           |
| token    | 10342978da134ae5bbb7dc94d2ba9c08 |
| expiry   | 2017-09-29T14:31:20.799212Z      |
+----------+----------------------------------+
```
If the username and password are valid and correctly entered in errbot's configuration file, errbot
will be authorised to interact with StackStorm's API/Stream end points.

#### user token

Test the errbot user token from the configuration using the `st2` command.  Make
sure no environment variables are set that could provide a valid token or api key already.

```
$ st2 action-alias list -t 10342978da134ae5bbb7dc94d2ba9c08
+-----------------------------------+------------+---------------------------------------+---------+
| ref                               | pack       | description                           | enabled |
+-----------------------------------+------------+---------------------------------------+---------+
| packs.pack_get                    | packs      | Get information about installed       | True    |
|                                   |            | StackStorm pack.                      |         |
| packs.pack_install                | packs      | Install/upgrade StackStorm packs.     | True    |
| packs.pack_search                 | packs      | Search for packs in StackStorm        | True    |
|                                   |            | Exchange and other directories.       |         |
| packs.pack_show                   | packs      | Show information about the pack from  | True    |
+-----------------------------------+------------+---------------------------------------+---------+
```
If a list of action aliases are shown, the token is valid.

#### api key
Confirm the api key has been created and still registered with StackStorm by using it with the `st2` command.
```
$ st2 apikey list --api-key ZzVk3DEBZ4FiZmMEmDBkM2x5ZmM5jWZkZWZjZjZmMZEwYzQwZD2iYzUyM2RhYTkTNMYmNDYNODIOOTYwMzE20A
+--------------------------+--------+-------------------------------------------+
| id                       | user   | metadata                                  |
+--------------------------+--------+-------------------------------------------+
| 586e6deadbeef66deadbeef6 | errbot | {u'used_by': u'errbot api access'}        |
+--------------------------+--------+-------------------------------------------+
```


### Is Errbot connected correctly to the chat back-end?
How to test if the bot is connected to the chat back-end is dependant on the back-end.  The simplest way is to send a message to the bot user account requesting the built in help.

E.g. Using a slack client the following command would be used
```/msg @bot_name !help```.

The bot should respond with its help text.

```
bot [11:01 AM]
_All commands_

*Backup*
_Backup related commands._
• *.backup* - Backup everything.
*ChatRoom*
_This is a basic implementation of a chatroom_
• *.room join* - Join (creating it first if needed) a chatroom.
• *.room occupants* - List the occupants in a given chatroom.
• *.room invite* - Invite one or more people into a chatroom.
• *.room topic* - Get or set the topic for a room.
```

### Is the StackStorm chatops pack installed and configured correctly?
Err-stackstorm requires the chatops pack to be installed.  To confirm it is installed, use the st2 cli.

```
$ st2 pack list
+-------------------+-------------------+--------------------------------+---------+----------------------+
| ref               | name              | description                    | version | author               |
+-------------------+-------------------+--------------------------------+---------+----------------------+
| chatops           | chatops           | Chatops integration pack       | 0.2.0   | Kirill Enykeev       |
```

Confirm the `notify_errbot.yaml` is inside the `chatops/rules` directory
```
$ cat /opt/stackstorm/packs/chatops/rules/notify_errbot.yaml
---
name: "notify-errbot"
pack: "chatops"
enabled: true
description: "Notification rule to send results of action executions to stream for chatops"
trigger:
  type: "core.st2.generic.notifytrigger"
criteria:
  trigger.route:
    pattern: "errbot"
    type: "equals"
action:
  ref: chatops.post_result
  parameters:
    channel: "{{ trigger.data.source_channel }}"
    user: "{{ trigger.data.user }}"
    execution_id: "{{ trigger.execution_id }}"
```

The rule should be available via the st2 command `st2 rule get chatops.notify-errbot`

```
+-------------+--------------------------------------------------------------+
| Property    | Value                                                        |
+-------------+--------------------------------------------------------------+
| id          | 5a6b1abc5b3a0f0f5bcd54e7                                     |
| uid         | rule:chatops:notify-errbot                                   |
| ref         | chatops.notify-errbot                                        |
| pack        | chatops                                                      |
| name        | notify-errbot                                                |
| description | Notification rule to send results of action executions to    |
|             | stream for chatops                                           |
| enabled     | True                                                         |
| action      | {                                                            |
|             |     "ref": "chatops.post_result",                            |
|             |     "parameters": {                                          |
|             |         "user": "{{trigger.data.user}}",                     |
|             |         "execution_id": "{{trigger.execution_id}}",          |
|             |         "channel": "{{trigger.data.source_channel}}"         |
|             |     }                                                        |
|             | }                                                            |
| criteria    | {                                                            |
|             |     "trigger.route": {                                       |
|             |         "pattern": "errbot",                                 |
|             |         "type": "equals"                                     |
|             |     }                                                        |
|             | }                                                            |
| tags        |                                                              |
| trigger     | {                                                            |
|             |     "type": "core.st2.generic.notifytrigger",                |
|             |     "ref": "core.st2.generic.notifytrigger",                 |
|             |     "parameters": {}                                         |
|             | }                                                            |
| type        | {                                                            |
|             |     "ref": "standard",                                       |
|             |     "parameters": {}                                         |
|             | }                                                            |
+-------------+--------------------------------------------------------------+
```


### Are events being sent via the StackStorm Stream?

From the errbot host connect to the StackStorm stream endpoint and watch for events emitted as actions
are executed by StackStorm.

```
curl -s -v -H 'Accept: text/event-stream' -H 'X-Auth-Token: 10342978da134ae5bbb7dc94d2ba9c08' http://<stackstorm_host>/stream/v1
```

### Are the events seen in the errbot logs using `errbot` as their route?

To see the events in the log, the debug level `BOT_LOG_LEVEL = logging.DEBUG` will need to be added to errbot's configuration file `config.py`.

If events are configured correctly, logs will be shown like this (`st2.announcement__errbot`)
```
17:04:12 DEBUG    root                      Dispatching st2.announcement__errbot event, 990 bytes...
17:04:12 DEBUG    lib.st2pluginapi          *** Errbot announcement event detected! ***
st2.announcement__errbot event, 990 bytes
```

If the announcement event is showing as
```
2018-01-26 15:51:55,246 DEBUG    sseclient                 Dispatching st2.announcement__chatops event, 508 bytes...
```
This indicates that the route wasn't set to `errbot`, see the Install Chatops section.
