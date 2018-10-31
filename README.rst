Discord Bot
===========

|Python| |Codacy Badge| |Maintainability|

A Discord Bot based on asynchronous programming (asyncio)

-  Notify twitch streams status
-  Dab
-  Ori and the Blind Forest Randomizer seed generation

Create a Twitch account to get a twich token
--------------------------------------------

Create a Twitch account: https://www.twitch.tv

Connect to twitch using the bot account and generate a chat token for
the bot

https://www.twitchtools.com/chat-token

Setup environment (Python 3.6+ required)
----------------------------------------

Requires 3.6 because: \* it uses ``async`` and ``await``, only available
for Python 3.5+ \* it uses fstrings, only available for Python 3.6+

Windows
~~~~~~~

::

    cd <project folder>
    virtualenv -p python3.6 .venv
    .venv/Script/pip.exe install -r requirements.txt

Linux
~~~~~

::

    cd <project folder>
    virtualenv -p python3.6 .venv
    .venv/bin/pip install -r requirements.txt

Create a database
-----------------

Create a postgresSQL database. The tables will be generated
automatically.

Create a configuration file
---------------------------

Create a first file ``config.yaml`` that will store all the unique bot
variables

.. code:: yaml

    COMMAND_PREFIX: "!"

    DISCORD_BOT_TOKEN: <discord bot token>
    TWITCH_API_CLIENT_ID: <twitch client id>

    RANDO_ROLE: "Looking For Rando"

    DATABASE_CREDENTIALS:
      host: <DB_HOST>
      port: <DB_PORT>
      database: <DB_NAME>
      user: <DB_USER>
      password: <DB_PASSWORD>

A second file ``default.yaml`` that will store all the default values

.. code:: yaml

    EXTENSIONS:
     - "stream"
     - "dab"
     - "ori_rando_seedgen"
     - "ori_logic_helper"

And a file for each guild the bot needs to behave differently. Allows to
set variable that only exist at a guild level like admin roles

.. code:: yaml

    GUILD_ID: <guild_id>

    ADMIN_ROLES:
     - "admin_role_1"
     - "admin_role_2"
     ...
     
    EXTENSIONS:
     - "<allowed cog>"
     - "<allowed cog>"
     ...

These values will override the default ones for that specific guild.

Run the bot
-----------

In the project folder, run:

Windows
~~~~~~~

::

    .venv/Script/python.exe main.py --config-dir /path/to/your/config/folder --log-dir /path/to/the/log/folder

Linux
~~~~~

::

    .venv/bin/python main.py --config-dir /path/to/your/config/folder --log-dir /path/to/the/log/folder

COGS
----

Twitch
~~~~~~

The Twitch cog allows you to track a list of streams. When one of the
streams goes online, the bot sends an embed message including the stream
information (game, title, ...)

Commands
^^^^^^^^

::

    # Display a list of the tracked streams
    !stream list

    # Add a stream in the tracked list
    !stream add <username>

    # Add a stream in the tracked list (the notification will include the tag @everyone)
    !stream everyone <username>

    # The notification will be sent the channel in which the command has been used

    # Remove a stream from the tracked list
    !stream remove <username>

How does it work ?
^^^^^^^^^^^^^^^^^^

Track streams
'''''''''''''

When an user uses whether ``!stream add`` or ``!stream everyone``, the
bot stores the twitch ``username`` as well as the discord channel
information in which the command has been called. Then, the bot requests
Twitch the twitch ``id`` for this ``username``. This ``id`` will be used
to retrieve the stream status.

Retrieving stream status
''''''''''''''''''''''''

The bot requests Twitch every X seconds using all the twitch ``ids``
previously added.

-  If stream was previously offline and goes online, the bot sends a
   notification in the related discord channel
-  If the stream was previously online and goes offline, the bot flags
   the stream as offline.

Troubleshooting
^^^^^^^^^^^^^^^

API request fails
'''''''''''''''''

The Twitch API returns a list of json object for each online stream.

If no stream is online, the API will return:

::

    {'stream': [] }

If the API call fails, the API will not return anything

::

    None

The bot then has to handle both of these case separately in order not to
tag all the streams as offline. Otherwise, the bot will notify every
stream again on the next successful API request.

Fake API responses
''''''''''''''''''

The Twitch API isn't implemented on an unique server. Depending on which
server the bot requests, the response can differ.

When a stream goes offline, we can see something like this happen

-  An API server sees the stream as offline
-  Another API server still sees the stream online several seconds after
   it went offline

If the bot requests the first server, it will find that the stream just
went offline. If the bot requests the second on the next API call, it
will understand that the stream just went online again and notify it.

To avoid that, the bot will look at the date when the stream went
offline. If the API still returns that the stream is offline during the
X next seconds, the bot tags it as offline. Otherwise it considers that
the stream didn't really go offline.

Ori and the Blind Forest seed generation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The seed generation allows you to generate a customized seed for the Ori
and the Blind Forest Randomizer, originally made by Sigmasin
https://github.com/sigmasin/OriDERandomizer

The bot uploads the ``randomizer.dat`` and the ``spoiler.txt`` in the
channel where the command is used

Commands
^^^^^^^^

::

    !seed [args...]

    Valid logics: casual, standard, expert, master, hard, ohko, 0xp, glitched
    Valid modes: shards, limitkeys, clues
    Valid flags: easy-path, normal-path, hard-path, normal, speed, dbash, extended, extended-damage, lure,
                 speed-lure, lure-hard, dboost, dboost-light, dboost-hard, cdash, cdash-farming, extreme,
                 timed-level, glitched

How does it work ?
^^^^^^^^^^^^^^^^^^

The bot requests the seed generator made by Eiko
https://github.com/turntekGodhead/ori\_coop\_server using the parameters
of the command then save the data in the seed and the spoiler.

Once the files are successfully downloaded, they are uploaded in the
channel where the command has been called.

Since the randomizer expects the seed file to have the name
``randomizer.dat`` to work, the bot creates a temporary subfolder in
which it downloads both files to avoid any name conflict. This subfolder
is deleted as soon as the files are sent in Discord.

.. |Python| image:: https://img.shields.io/badge/Python-3.6%2B-blue.svg
   :target: https://www.python.org/
.. |Codacy Badge| image:: https://api.codacy.com/project/badge/Grade/902886185fd2476dadab0cb1a4c4f3a9
   :target: https://app.codacy.com/app/tesence/discord_bot?utm_source=github.com&utm_medium=referral&utm_content=tesence/discord_bot&utm_campaign=Badge_Grade_Dashboard
.. |Maintainability| image:: https://api.codeclimate.com/v1/badges/e5874485dd3795f5e940/maintainability
   :target: https://codeclimate.com/github/tesence/discord_bot/maintainability