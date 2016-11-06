# dokku mariadb (beta) [![Build Status](https://img.shields.io/travis/dokku/dokku-mariadb.svg?branch=master "Build Status")](https://travis-ci.org/dokku/dokku-mariadb) [![IRC Network](https://img.shields.io/badge/irc-freenode-blue.svg "IRC Freenode")](https://webchat.freenode.net/?channels=dokku)

Official mariadb plugin for dokku. Currently defaults to installing [mariadb 10.1.16](https://hub.docker.com/_/mariadb/).

## requirements

- dokku 0.4.x+
- docker 1.8.x

## installation

```shell
# on 0.4.x+
sudo dokku plugin:install https://github.com/dokku/dokku-mariadb.git mariadb
```

## commands

```
mariadb:backup <name> <bucket>   Create a backup of the mariadb service to an existing s3 bucket
mariadb:backup-auth <name> <aws_access_key_id> <aws_secret_access_key> Sets up authentication for backups on the mariadb service
mariadb:backup-deauth <name>     Removes backup authentication for the mariadb service
mariadb:backup-schedule <name> <schedule>  <aws_access_key_id> <aws_secret_access_key> <bucket> Schedules a backup of the mariadb service
mariadb:backup-unschedule <name> Unschedules the backup of the mariadb service
mariadb:clone <name> <new-name>  Create container <new-name> then copy data from <name> into <new-name>
mariadb:connect <name>           Connect via mariadb to a mariadb service
mariadb:create <name>            Create a mariadb service with environment variables
mariadb:destroy <name>           Delete the service and stop its container if there are no links left
mariadb:enter <name> [command]   Enter or run a command in a running mariadb service container
mariadb:export <name> > <file>   Export a dump of the mariadb service database
mariadb:expose <name> [port]     Expose a mariadb service on custom port if provided (random port otherwise)
mariadb:import <name> < <file>   Import a dump into the mariadb service database
mariadb:info <name>              Print the connection information
mariadb:link <name> <app>        Link the mariadb service to the app
mariadb:list                     List all mariadb services
mariadb:logs <name> [-t]         Print the most recent log(s) for this service
mariadb:promote <name> <app>     Promote service <name> as DATABASE_URL in <app>
mariadb:restart <name>           Graceful shutdown and restart of the mariadb service container
mariadb:start <name>             Start a previously stopped mariadb service
mariadb:stop <name>              Stop a running mariadb service
mariadb:unexpose <name>          Unexpose a previously exposed mariadb service
mariadb:unlink <name> <app>      Unlink the mariadb service from the app
```

## usage

```shell
# create a mariadb service named lolipop
dokku mariadb:create lolipop

# you can also specify the image and image
# version to use for the service
# it *must* be compatible with the
# official mariadb image
export MARIADB_IMAGE="mariadb"
export MARIADB_IMAGE_VERSION="5.5"
dokku mariadb:create lolipop

# you can also specify custom environment
# variables to start the mariadb service
# in semi-colon separated forma
export MARIADB_CUSTOM_ENV="USER=alpha;HOST=beta"
dokku mariadb:create lolipop

# get connection information as follows
dokku mariadb:info lolipop

# you can also retrieve a specific piece of service info via flags
dokku mariadb:info lolipop --config-dir
dokku mariadb:info lolipop --data-dir
dokku mariadb:info lolipop --dsn
dokku mariadb:info lolipop --exposed-ports
dokku mariadb:info lolipop --id
dokku mariadb:info lolipop --internal-ip
dokku mariadb:info lolipop --links
dokku mariadb:info lolipop --service-root
dokku mariadb:info lolipop --status
dokku mariadb:info lolipop --version

# a bash prompt can be opened against a running service
# filesystem changes will not be saved to disk
dokku mariadb:enter lolipop

# you may also run a command directly against the service
# filesystem changes will not be saved to disk
dokku mariadb:enter lolipop ls -lah /

# a mariadb service can be linked to a
# container this will use native docker
# links via the docker-options plugin
# here we link it to our 'playground' app
# NOTE: this will restart your app
dokku mariadb:link lolipop playground

# the following environment variables will be set automatically by docker (not
# on the app itself, so they won’t be listed when calling dokku config)
#
#   DOKKU_MARIADB_LOLIPOP_NAME=/lolipop/DATABASE
#   DOKKU_MARIADB_LOLIPOP_PORT=tcp://172.17.0.1:3306
#   DOKKU_MARIADB_LOLIPOP_PORT_3306_TCP=tcp://172.17.0.1:3306
#   DOKKU_MARIADB_LOLIPOP_PORT_3306_TCP_PROTO=tcp
#   DOKKU_MARIADB_LOLIPOP_PORT_3306_TCP_PORT=3306
#   DOKKU_MARIADB_LOLIPOP_PORT_3306_TCP_ADDR=172.17.0.1
#
# and the following will be set on the linked application by default
#
#   DATABASE_URL=mysql://mariadb:SOME_PASSWORD@dokku-mariadb-lolipop:3306/lolipop
#
# NOTE: the host exposed here only works internally in docker containers. If
# you want your container to be reachable from outside, you should use `expose`.

# another service can be linked to your app
dokku mariadb:link other_service playground

# since DATABASE_URL is already in use, another environment variable will be
# generated automatically
#
#   DOKKU_MARIADB_BLUE_URL=mysql://mariadb:ANOTHER_PASSWORD@dokku-mariadb-other-service:3306/other_service

# you can then promote the new service to be the primary one
# NOTE: this will restart your app
dokku mariadb:promote other_service playground

# this will replace DATABASE_URL with the url from other_service and generate
# another environment variable to hold the previous value if necessary.
# you could end up with the following for example:
#
#   DATABASE_URL=mysql://mariadb:ANOTHER_PASSWORD@dokku-mariadb-other_service:3306/other_service
#   DOKKU_MARIADB_BLUE_URL=mysql://mariadb:ANOTHER_PASSWORD@dokku-mariadb-other-service:3306/other_service
#   DOKKU_MARIADB_SILVER_URL=mysql://mariadb:SOME_PASSWORD@dokku-mariadb-lolipop:3306/lolipop

# you can also unlink a mariadb service
# NOTE: this will restart your app and unset related environment variables

# you can tail logs for a particular service
dokku mariadb:logs lolipop
dokku mariadb:logs lolipop -t # to tail

# you can dump the database
dokku mariadb:export lolipop > lolipop.sql

# you can import a dump
dokku mariadb:import lolipop < database.sql

# you can clone an existing database to a new one
dokku mariadb:clone lolipop new_database

# finally, you can destroy the container
dokku mariadb:destroy lolipop
```

## Changing database adapter

It's possible to change the protocol for DATABASE_URL by setting
the environment variable MARIADB_DATABASE_SCHEME on the app:

```
dokku config:set playground MARIADB_DATABASE_SCHEME=mariadb2
dokku mariadb:link lolipop playground
```

Will cause DATABASE_URL to be set as
mariadb2://mariadb:SOME_PASSWORD@dokku-mariadb-lolipop:3306/lolipop

CAUTION: Changing MARIADB_DATABASE_SCHEME after linking will cause dokku to
believe the mariadb is not linked when attempting to use `dokku mariadb:unlink`
or `dokku mariadb:promote`.
You should be able to fix this by

- Changing MARIADB_URL manually to the new value.

OR

- Set MARIADB_DATABASE_SCHEME back to its original setting
- Unlink the service
- Change MARIADB_DATABASE_SCHEME to the desired setting
- Relink the service

## Backups

Datastore backups are supported via AWS S3. The only supported region is `us-east-1`, and using an S3 bucket in another region will result in an error. If you would like to sponsor work to enable support for other regions, please contact @josegonzalez.

Backups can be performed using the backup commands:

```
# setup s3 backup authentication
dokku mariadb:backup-auth lolipop AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY

# remove s3 authentication
dokku mariadb:backup-deauth lolipop

# backup the `lolipop` service to the `BUCKET_NAME` bucket on AWS
dokku mariadb:backup lolipop BUCKET_NAME

# schedule a backup
dokku mariadb:backup-schedule lolipop CRON_SCHEDULE BUCKET_NAME

# remove the scheduled backup from cron
dokku mariadb:backup-unschedule lolipop
```
