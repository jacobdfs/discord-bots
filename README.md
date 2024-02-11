# What is this?

This an open source discord bot which assists with game matchmaking.

This uses Microsoft's [Trueskill](https://www.microsoft.com/en-us/research/project/trueskill-ranking-system/)
matchmaking algorithm to create fair and
balanced teams. Players are assigned a level of skill (mu) and uncertainty (sigma) which is adjusted after each game.

Games are organized using a queueing system and the bot supports features like segmentation by region, game type, ranked / unranked, map rotations, voice channel creation, leaderboards, Twitch integration, dice rolls, coin
flips, raffles, commends, post-game images, and more.

## Installation

The bot is written in Python 3 and uses sqlite for persistence. You will need a `DISCORD_API_KEY` at a minimum to run
the bot.

### MacOS

1. Install Python 3.10.0: https://docs.python-guide.org/starting/install3/osx/

    - (optional) Install Python with pyenv instead:
    - `brew install pyenv`
    - `pyenv install 3.10.0`
    - `pyenv global 3.10.0`

1. Setup a virtual env:
    - `cd discord-bots`
    - `python3 -m venv .venv`
    - `source .venv/bin/activate`
1. `pip install -U .`
1. `cp .env.example .env`. Modify `.env` by adding your API key
1. Setup the database: `alembic upgrade head`

### Linux

TODO

### Windows

1. Install WSL
1. Install Python
1. Set up a virtualenv

- `python3 -m venv .venv`
- `.venv\Scripts\activate`. If you see this error:
  ```
  File <>\discord-bots\.venv\Scripts\Activate.ps1 cannot be loaded because running scripts is disabled on this system. For more information, see about_Execution_Policies at https:/go.microsoft.com/fwlink/?LinkID=135170.
  ```
  You may need to adjust your windows execution policy: https://stackoverflow.com/a/18713789

## .env file configuration

The following are required

- `DISCORD_API_KEY`
- `CHANNEL_ID` - The discord id of the channel the bot will live in
- `DATABASE_URI` -
- `TRIBES_VOICE_CATEGORY_CHANNEL_ID` - The id of the voice channel category (so the bot can make voice channels)
- `SEED_ADMIN_IDS` - Discord ids of players that will start off as admin. You'll need at least one in order to create
  more
- `DATABASE_URI` - URL to the database

To find some of these values, set `ENABLE_DEBUG=True` in `.env`, then in any channel type `!configure`. This will print to console your ID, the channel ID, and voice channel category IDs.

The following are optional

- `TWITCH_GAME_NAME`, `TWITCH_CLIENT_ID`, `TWITCH_CLIENT_SECRET` - These enable the `streams` command to list current
  streams of the specified game
- `COMMAND_PREFIX` - Use a different prefix instead of `!`
- `DEFAULT_TRUESKILL_MU`, `DEFAULT_TRUESKILL_SIGMA` - Customize the default trueskill for new players. Handle with care!
- `SHOW_TRUESKILL` - Shows player trueskill when making teams, enables the trueskill leaderboard, etc.
- `REQUIRE_ADD_TARGET` - Players have to specify a queue to add to
- `ENABLE_VOICE_MOVE` - Enables moving players between voice channels
- `DEFAULT_VOICE_MOVE` - Sets default move behaviour. True will auto opt-in new players. Defaults to False
- `ALLOW_VULGAR_NAMES` - Allow dirtier team names. Defaults to False
- `ENABLE_DEBUG` - Defaults to False
- `ENABLE_RAFFLE` - Defaults to False
- `DEFAULT_RAFFLE_VALUE` - Defaults to 5
- `SHOW_LEFT_RIGHT_TEAM` - Add an indicator which team to add to (`(L)` or `(R)`)
- `MAXIMUM_TEAM_COMBINATIONS` - Maximum amount of team combinations to try. Performance optimization for larger game modes like 12v12 at the cost of potentially not finding the ideal game.
- `LEADERBOARD_CHANNEL`
- `RE_ADD_DELAY` - Time after votes or games finish until the add is processed to allow equal access to all players, so they don't have to exit their game early to participate in the next one.
- `AFK_TIME_MINUTES` - If no activity is registered within this time, the player is automatically deleted from all queues and votes are removed. Defaults to 45.
- `MAP_ROTATION_MINUTES` - Time since last map switch after which the maps auto-rotates. Defaults to 60.
- `DISABLE_MAP_ROTATION`
- `MAP_VOTE_THRESHOLD` - Defaults to 7. The number of votes needed to succeed a map skip / replacement
- `STATS_DIR` - Defaults to None. Store screenshots if configured. Doesn't work on Windows.
- `STATS_WIDTH`, `STATS_HEIGHT`
- `CURRENCY_NAME` - Set's the currency name for player economy. Default is Shazbucks
- `STARTING_CURRENCY` - Define a customer starting currency value for new players. Default is 100

## Running the bot

1. `cd discord-bots`
1. `source .venv/bin/activate`
2. `alembic upgrade head`
1. `python -m discord_bots.main`

### Pulling in updates

1. `git pull`
2. `alembic upgrade head`
3. Restart the python process

# Development

The bot is written in Python 3 with types as much as possible (enforced by Pylance). We use SQLAlchemy as the ORM
and `alembic` to handle migrations.

## Installation

Postgres is the new preferred way of doing development. To install `psycopg2` you'll need to install `libpq`.

### Ubuntu
```
sudo apt-get install libpq-dev
```

### Arch linux
```
pacman -S postgresql-libs
```

The steps are the same but use `pip install -e .` instead. This allows local changes to be picked up automatically.

## Database

Install Docker
```
sudo snap install docker
```

After installation, start a local database with:
```
sudo docker run --name postgres -e POSTGRES_PASSWORD=password -d -p 5432:5432 postgres
```

To restart an existing docker container, use:
```
sudo docker restart postgres
```

Install postgresql-client
```
sudo apt-get install postgresql-client
```

To connect with PSQL:
```
PGPASSWORD=password psql -h localhost -U postgres -d postgres -p 5432
```

In `.env`:
```
DATABASE_URI=postgresql://postgres:password@localhost:5432/postgres
```

## Editor

Recommend using vscode. If you do, install these vscode plugins:

- Python
- Pylance

## Type checking

If you use vscode add this to your settings.json (if anyone knows how to commit
this to the project lmk!):
https://www.emmanuelgautier.com/blog/enable-vscode-python-type-checking

```json
{
  "python.analysis.typeCheckingMode": "basic"
}
```

This enforces type checks for the types declared

## Formatting

Use python black: https://github.com/psf/black

- Go to vscode preferences (cmd + `,` on mac, ctrl + `,` on windows)
- Type "python formatting" in the search bar
- For the option `Python > Formatting: Provider` select `black`

### Pre-commit hook

This project uses `darker` for formatting in a pre-commit hook. Darker documentation: https://pypi.org/project/darker/. pre-commit documentation: https://pre-commit.com/#installation

- `pip install darker`
- `pip install pre-commit`
- `pre-commit install`
- `pre-commit autoupdate`

## Migrations

Migrations are handled by Alembic: https://alembic.sqlalchemy.org/. See here for a
tutorial: https://alembic.sqlalchemy.org/en/latest/tutorial.html.

To apply migrations:

- `alembic upgrade head`

To create new migrations:

- Make your changes in `models.py`
- Generate a migration file: `alembic revision --autogenerate -m "Your migration name here"`. Your migration file will
  be in `alembic/versions`.
- Apply your migration to the database: `alembic upgrade head`
- Commit your migration: `git add alembic/versions`

Common issues:

- Alembic does not pick up certain changes like renaming tables or columns
  correctly. For these changes you'll need to manually edit the migration file.
  See here for a full list of changes Alembic will not detect correctly:
  https://alembic.sqlalchemy.org/en/latest/autogenerate.html#what-does-autogenerate-detect-and-what-does-it-not-detect
- To set a default value for a column, you'll need to use `server_default`:
  https://docs.sqlalchemy.org/en/14/core/defaults.html#server-defaults. This sets
  a default on the database side.
- Alembic also sometimes has issues with constraints and naming. If you run into
  an issue like this, you may need to hand edit the migration. See here:
  https://alembic.sqlalchemy.org/en/latest/naming.html

# Full list of commands

```
CategoryCommands:
  clearqueuecategory
  createcategory
  listcategories
  removecategory
  setcategoryname
  setcategoryrated
  setcategoryunrated
  setqueuecategory
MapCommands:
  addmap                  Add a map to the map pool
  changegamemap           Change the map for a game
  changequeuemap          Change the next map for a queue (note: affects all queues sharing that rotation)
  listmaps                List all maps in the map pool
  removemap               Remove a map from the map pool
QueueCommands:
  setqueuerotation        Assign a map rotation to a queue
  showqueuerotation       Shows the map rotation assigned to a queue
RaffleCommands:
  createraffle            TODO: Implementation
  myraffle                Displays how many raffle tickets you have
  rafflestatus            Displays raffle ticket information and raffle leaderboard
  runraffle               TODO: Implementation
  setrotationmapraffle    Set the raffle ticket reward for a map in a rotation
RotationCommands:
  addrotation             Add a rotation to the rotation pool
  addrotationmap          Add a map to a rotation at a specific ordinal (position)
  listrotations           List all rotations in the rotation pool
  removerotation          Remove a rotation from the rotation pool
  removerotationmap       Remove a map from a rotation
  setrotationmapordinal   Set the ordinal (position) for a map in a rotation
VoteCommands:
  mockvotes               Generates 6 mock votes for testing
  setmapvotethreshold     Set the number of votes required to pass
  unvote                  Remove all of a player's votes
  unvotemap               Remove all of a player's votes for a map
  unvoteskip              Remove all of a player's votes to skip the next map
​No Category:
  add                     Players adds self to queue(s). If no args to all existing queues
  addadmin
  addadminrole
  addqueuerole
  autosub                 Picks a person to sub at random
  ban                     TODO: remove player from queues
  cancelgame
  clearqueue
  clearqueuerange
  coinflip
  commend
  commendstats
  createcommand
  createdbbackup
  createqueue
  decayplayer
  del                     Players deletes self from queue(s)
  deletegame
  delplayer               Admin command to delete player from all queues
  disableleaderboard
  disablestats
  editcommand
  editgamewinner
  enableleaderboard
  enablestats
  finishgame
  gamehistory
  help                    Shows this message
  isolatequeue
  listadminroles
  listadmins
  listbans
  listchannels
  listdbbackups
  listnotifications
  listplayerdecays
  listqueueroles
  lockqueue
  lt
  mockqueue
  movegameplayers
  notify
  pug
  removeadmin
  removeadminrole
  removecommand
  removedbbackup
  removenotifications
  removequeue
  removequeuerole
  resetleaderboardchannel
  resetplayertrueskill
  restart
  roll
  setbias
  setcaptainbias
  setcommandprefix
  setgamecode
  setqueueordinal
  setqueuerange
  setqueuesweaty
  setsigma
  showgame
  showgamedebug
  showqueuerange
  showsigma               Returns the player's base sigma. Doesn't consider regions
  showtrueskillnormdist   Print the normal distribution of the trueskill in a given queue.
  stats
  status
  streams
  sub
  testleaderboard
  trueskill
  unban
  unisolatequeue
  unlockqueue
  unsetqueuesweaty
```
