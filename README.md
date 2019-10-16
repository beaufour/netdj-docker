# What is this
Host setup for Mopidy+Iris, Icecast, and a Mopidy Slack bot.

It will provide a full solution that allows you to play Spotify music over the web (through Iris),
and then stream it (to multiple locations) via IceCast. It will also run a Slack bot that will post
the songs being played in a channel, and allow up to ask for the queue and skip the current number.

It consists of four docker containers. One for each of the services above and then Nginx in
front which will expose `/iris/` and `/stream/`.

# How to use

## Initial Setup
1. get host set up with `docker`, `docker-compose`, `git`, `node`, `htpasswd`, and `npm`.
2. `mkdir netdj`
3. `cd netdj`
4. `git clone THIS REPO`
5. `git clone MOPIDY_BOT_REPO`
6. link or put your music in `music`
7. link or put your playlists in `playlists`
8. `mkdir netdj-configs`
9. copy the `.sample` files into `netdj-configs`, remove the `.sample` ending, and edit so they have your
   information.
10. run `htpasswd` and create web credentials and put the file in `netdj-configs/htpasswd`
11. inside `MOPIDY_BOT_REPO` run `npm install` (yes, should really be run inside the container...)
12. create Spotify credentials and place inside `netdj-configs/mopidy.conf`
13. change Icecast credentials in `netdj-configs/icecast-env`
13. set Icecast source password in the audio/output line in `netdj-configs/mopidy.conf`

Now you should be ready to run `docker-compose up` inside `THIS REPO`. Watch the log output and see
if everything went as planned. Repeat `docker-compose down`, fix stuff, and `docker-compose up`
until it works :) Once it does you should be able to go to `http://HOST/iris/` and start playing
stuff, and then you can stream from `http://HOST/stream/`.

The last setup step is to get the bot connected. You need to create a bot on slack, to get the
credentials (to put in `bot-env`). The "Request URL" is `http://HOST/bot/slack/events`. You also
need to specify a channel ID for where the bot posts the songs that are played (in `bot-env` too).

## Running

After the setup above, it is just a question of running `docker-compose up` (potentially with `-d`
to detach).

## Things that aren't ideal

* we want to add a source fallback with a silent mp3 to avoid the stream stopping when skipping tracks,
  but that means we have to override all of the icecast configuration and thus setting passwords etc does
  not work anymore (with the stock docker image). We should really create our own docker image...

* certs are hosted on the host and not in a container. When using letsencrypt maybe it should live in the
  web service itself, or maybe a separate container?

* nginx prod vs debug handling of certs is ugly
