# dokku mariadb [![Build Status](https://img.shields.io/travis/dokku/dokku-mariadb.svg?branch=master "Build Status")](https://travis-ci.org/dokku/dokku-mariadb) [![IRC Network](https://img.shields.io/badge/irc-freenode-blue.svg "IRC Freenode")](https://webchat.freenode.net/?channels=dokku)

Official mariadb plugin for dokku. Currently defaults to installing [mariadb 10.4.10](https://hub.docker.com/_/mariadb/).

## Requirements

- dokku 0.12.x+
- docker 1.8.x

## Installation

```shell
# on 0.12.x+
sudo dokku plugin:install https://github.com/dokku/dokku-mariadb.git mariadb
```

## Commands

```
mariadb:app-links <app>                            # list all mariadb service links for a given app
mariadb:backup <service> <bucket-name> [--use-iam] # creates a backup of the mariadb service to an existing s3 bucket
mariadb:backup-auth <service> <aws-access-key-id> <aws-secret-access-key> <aws-default-region> <aws-signature-version> <endpoint-url> # sets up authentication for backups on the mariadb service
mariadb:backup-deauth <service>                    # removes backup authentication for the mariadb service
mariadb:backup-schedule <service> <schedule> <bucket-name> [--use-iam] # schedules a backup of the mariadb service
mariadb:backup-schedule-cat <service>              # cat the contents of the configured backup cronfile for the service
mariadb:backup-set-encryption <service> <passphrase> # sets encryption for all future backups of mariadb service
mariadb:backup-unschedule <service>                # unschedules the backup of the mariadb service
mariadb:backup-unset-encryption <service>          # unsets encryption for future backups of the mariadb service
mariadb:clone <service> <new-service> [--clone-flags...] # create container <new-name> then copy data from <name> into <new-name>
mariadb:connect <service>                          # connect to the service via the mariadb connection tool
mariadb:create <service> [--create-flags...]       # create a mariadb service
mariadb:destroy <service> [-f|--force]             # delete the mariadb service/data/container if there are no links left
mariadb:enter <service>                            # enter or run a command in a running mariadb service container
mariadb:exists <service>                           # check if the mariadb service exists
mariadb:export <service>                           # export a dump of the mariadb service database
mariadb:expose <service> <ports...>                # expose a mariadb service on custom port if provided (random port otherwise)
mariadb:import <service>                           # import a dump into the mariadb service database
mariadb:info <service> [--single-info-flag]        # print the connection information
mariadb:link <service> <app> [--link-flags...]     # link the mariadb service to the app
mariadb:linked <service> <app>                     # check if the mariadb service is linked to an app
mariadb:links <service>                            # list all apps linked to the mariadb service
mariadb:list                                       # list all mariadb services
mariadb:logs <service> [-t|--tail]                 # print the most recent log(s) for this service
mariadb:promote <service> <app>                    # promote service <service> as DATABASE_URL in <app>
mariadb:restart <service>                          # graceful shutdown and restart of the mariadb service container
mariadb:start <service>                            # start a previously stopped mariadb service
mariadb:stop <service>                             # stop a running mariadb service
mariadb:unexpose <service>                         # unexpose a previously exposed mariadb service
mariadb:unlink <service> <app>                     # unlink the mariadb service from the app
mariadb:upgrade <service> [--upgrade-flags...]     # upgrade service <service> to the specified versions
```

## Usage

Help for any commands can be displayed by specifying the command as an argument to mariadb:help. Please consult the `mariadb:help` command for any undocumented commands.

### Basic Usage
### list all mariadb services

```shell
# usage
dokku mariadb:list 
```

examples:

List all services:

```shell
dokku mariadb:list
```
### create a mariadb service

```shell
# usage
dokku mariadb:create <service> [--create-flags...]
```

examples:

Create a mariadb service named lolipop:

```shell
dokku mariadb:create lolipop
```

You can also specify the image and image version to use for the service. It *must* be compatible with the ${plugin_image} image. :

```shell
export DATABASE_IMAGE="${PLUGIN_IMAGE}"
export DATABASE_IMAGE_VERSION="${PLUGIN_IMAGE_VERSION}"
dokku mariadb:create lolipop
```

You can also specify custom environment variables to start the mariadb service in semi-colon separated form. :

```shell
export DATABASE_CUSTOM_ENV="USER=alpha;HOST=beta"
dokku mariadb:create lolipop
```
### print the connection information

```shell
# usage
dokku mariadb:info <service> [--single-info-flag]
```

examples:

Get connection information as follows:

```shell
dokku mariadb:info lolipop
```

You can also retrieve a specific piece of service info via flags:

```shell
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
```
### print the most recent log(s) for this service

```shell
# usage
dokku mariadb:logs <service> [-t|--tail]
```

examples:

You can tail logs for a particular service:

```shell
dokku mariadb:logs lolipop
```

By default, logs will not be tailed, but you can do this with the --tail flag:

```shell
dokku mariadb:logs lolipop --tail
```
### link the mariadb service to the app

```shell
# usage
dokku mariadb:link <service> <app> [--link-flags...]
```

examples:

A mariadb service can be linked to a container. This will use native docker links via the docker-options plugin. Here we link it to our 'playground' app. :

> NOTE: this will restart your app

```shell
dokku mariadb:link lolipop playground
```

The following environment variables will be set automatically by docker (not on the app itself, so they won’t be listed when calling dokku config):

```
DOKKU_DATABASE_LOLIPOP_NAME=/lolipop/DATABASE
DOKKU_DATABASE_LOLIPOP_PORT=tcp://172.17.0.1:3306
DOKKU_DATABASE_LOLIPOP_PORT_3306_TCP=tcp://172.17.0.1:3306
DOKKU_DATABASE_LOLIPOP_PORT_3306_TCP_PROTO=tcp
DOKKU_DATABASE_LOLIPOP_PORT_3306_TCP_PORT=3306
DOKKU_DATABASE_LOLIPOP_PORT_3306_TCP_ADDR=172.17.0.1
```

The following will be set on the linked application by default:

```
DATABASE_URL=mysql://lolipop:SOME_PASSWORD@dokku-mariadb-lolipop:3306/lolipop
```

The host exposed here only works internally in docker containers. If you want your container to be reachable from outside, you should use the 'expose' subcommand. Another service can be linked to your app:

```shell
dokku mariadb:link other_service playground
```

It is possible to change the protocol for database_url by setting the environment variable database_database_scheme on the app. Doing so will after linking will cause the plugin to think the service is not linked, and we advise you to unlink before proceeding. :

```shell
dokku config:set playground DATABASE_DATABASE_SCHEME=mysql2
dokku mariadb:link lolipop playground
```

This will cause database_url to be set as:

```
mysql2://lolipop:SOME_PASSWORD@dokku-mariadb-lolipop:3306/lolipop
```
### unlink the mariadb service from the app

```shell
# usage
dokku mariadb:unlink <service> <app>
```

examples:

You can unlink a mariadb service:

> NOTE: this will restart your app and unset related environment variables

```shell
dokku mariadb:unlink lolipop playground
```
### delete the mariadb service/data/container if there are no links left

```shell
# usage
dokku mariadb:destroy <service> [-f|--force]
```

examples:

Destroy the service, it's data, and the running container:

```shell
dokku mariadb:destroy lolipop
```

### Service Lifecycle

The lifecycle of each service can be managed through the following commands:

### connect to the service via the mariadb connection tool

```shell
# usage
dokku mariadb:connect <service>
```

examples:

Connect to the service via the mariadb connection tool:

```shell
dokku mariadb:connect lolipop
```
### enter or run a command in a running mariadb service container

```shell
# usage
dokku mariadb:enter <service>
```

examples:

A bash prompt can be opened against a running service. Filesystem changes will not be saved to disk. :

```shell
dokku mariadb:enter lolipop
```

You may also run a command directly against the service. Filesystem changes will not be saved to disk. :

```shell
dokku mariadb:enter lolipop touch /tmp/test
```
### expose a mariadb service on custom port if provided (random port otherwise)

```shell
# usage
dokku mariadb:expose <service> <ports...>
```

examples:

Expose the service on the service's normal ports, allowing access to it from the public interface (0. 0. 0. 0):

```shell
dokku mariadb:expose lolipop ${PLUGIN_DATASTORE_PORTS[@]}
```
### unexpose a previously exposed mariadb service

```shell
# usage
dokku mariadb:unexpose <service>
```

examples:

Unexpose the service, removing access to it from the public interface (0. 0. 0. 0):

```shell
dokku mariadb:unexpose lolipop
```
### promote service <service> as DATABASE_URL in <app>

```shell
# usage
dokku mariadb:promote <service> <app>
```

examples:

If you have a mariadb service linked to an app and try to link another mariadb service another link environment variable will be generated automatically:

```
DOKKU_DATABASE_BLUE_URL=mysql://other_service:ANOTHER_PASSWORD@dokku-mariadb-other-service:3306/other_service
```

You can promote the new service to be the primary one:

> NOTE: this will restart your app

```shell
dokku mariadb:promote other_service playground
```

This will replace database_url with the url from other_service and generate another environment variable to hold the previous value if necessary. You could end up with the following for example:

```
DATABASE_URL=mysql://other_service:ANOTHER_PASSWORD@dokku-mariadb-other-service:3306/other_service
DOKKU_DATABASE_BLUE_URL=mysql://other_service:ANOTHER_PASSWORD@dokku-mariadb-other-service:3306/other_service
DOKKU_DATABASE_SILVER_URL=mysql://lolipop:SOME_PASSWORD@dokku-mariadb-lolipop:3306/lolipop
```
### graceful shutdown and restart of the mariadb service container

```shell
# usage
dokku mariadb:restart <service>
```

examples:

Restart the service:

```shell
dokku mariadb:restart lolipop
```
### start a previously stopped mariadb service

```shell
# usage
dokku mariadb:start <service>
```

examples:

Start the service:

```shell
dokku mariadb:start lolipop
```
### stop a running mariadb service

```shell
# usage
dokku mariadb:stop <service>
```

examples:

Stop the service and the running container:

```shell
dokku mariadb:stop lolipop
```
### upgrade service <service> to the specified versions

```shell
# usage
dokku mariadb:upgrade <service> [--upgrade-flags...]
```

examples:

You can upgrade an existing service to a new image or image-version:

```shell
dokku mariadb:upgrade lolipop
```

### Service Automation

Service scripting can be executed using the following commands:

### list all mariadb service links for a given app

```shell
# usage
dokku mariadb:app-links <app>
```

examples:

List all mariadb services that are linked to the 'playground' app. :

```shell
dokku mariadb:app-links playground
```
### create container <new-name> then copy data from <name> into <new-name>

```shell
# usage
dokku mariadb:clone <service> <new-service> [--clone-flags...]
```

examples:

You can clone an existing service to a new one:

```shell
dokku mariadb:clone lolipop lolipop-2
```
### check if the mariadb service exists

```shell
# usage
dokku mariadb:exists <service>
```

examples:

Here we check if the lolipop mariadb service exists. :

```shell
dokku mariadb:exists lolipop
```
### check if the mariadb service is linked to an app

```shell
# usage
dokku mariadb:linked <service> <app>
```

examples:

Here we check if the lolipop mariadb service is linked to the 'playground' app. :

```shell
dokku mariadb:linked lolipop playground
```
### list all apps linked to the mariadb service

```shell
# usage
dokku mariadb:links <service>
```

examples:

List all apps linked to the 'lolipop' mariadb service. :

```shell
dokku mariadb:links lolipop
```

### Data Management

The underlying service data can be imported and exported with the following commands:

### import a dump into the mariadb service database

```shell
# usage
dokku mariadb:import <service>
```

examples:

Import a datastore dump:

```shell
dokku mariadb:import lolipop < database.dump
```
### export a dump of the mariadb service database

```shell
# usage
dokku mariadb:export <service>
```

examples:

By default, datastore output is exported to stdout:

```shell
dokku mariadb:export lolipop
```

You can redirect this output to a file:

```shell
dokku mariadb:export lolipop > lolipop.dump
```

### Backups

Datastore backups are supported via AWS S3 and S3 compatible services like [minio](https://github.com/minio/minio).

You may skip the `backup-auth` step if your dokku install is running within EC2 and has access to the bucket via an IAM profile. In that case, use the `--use-iam` option with the `backup` command.

Backups can be performed using the backup commands:

### sets up authentication for backups on the mariadb service

```shell
# usage
dokku mariadb:backup-auth <service> <aws-access-key-id> <aws-secret-access-key> <aws-default-region> <aws-signature-version> <endpoint-url>
```

examples:

Setup s3 backup authentication:

```shell
dokku mariadb:backup-auth lolipop AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY
```

Setup s3 backup authentication with different region:

```shell
dokku mariadb:backup-auth lolipop AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_REGION
```

Setup s3 backup authentication with different signature version and endpoint:

```shell
dokku mariadb:backup-auth lolipop AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_REGION AWS_SIGNATURE_VERSION ENDPOINT_URL
```

More specific example for minio auth:

```shell
dokku mariadb:backup-auth lolipop MINIO_ACCESS_KEY_ID MINIO_SECRET_ACCESS_KEY us-east-1 s3v4 https://YOURMINIOSERVICE
```
### removes backup authentication for the mariadb service

```shell
# usage
dokku mariadb:backup-deauth <service>
```

examples:

Remove s3 authentication:

```shell
dokku mariadb:backup-deauth lolipop
```
### creates a backup of the mariadb service to an existing s3 bucket

```shell
# usage
dokku mariadb:backup <service> <bucket-name> [--use-iam]
```

examples:

Backup the 'lolipop' service to the 'my-s3-bucket' bucket on aws:

```shell
dokku mariadb:backup lolipop my-s3-bucket --use-iam
```
### sets encryption for all future backups of mariadb service

```shell
# usage
dokku mariadb:backup-set-encryption <service> <passphrase>
```

examples:

Set a gpg passphrase for backups:

```shell
dokku mariadb:backup-set-encryption lolipop
```
### unsets encryption for future backups of the mariadb service

```shell
# usage
dokku mariadb:backup-unset-encryption <service>
```

examples:

Unset a gpg encryption key for backups:

```shell
dokku mariadb:backup-unset-encryption lolipop
```
### schedules a backup of the mariadb service

```shell
# usage
dokku mariadb:backup-schedule <service> <schedule> <bucket-name> [--use-iam]
```

examples:

Schedule a backup:

> 'schedule' is a crontab expression, eg. "0 3 * * *" for each day at 3am

```shell
dokku mariadb:backup-schedule lolipop "0 3 * * *" my-s3-bucket
```

Schedule a backup and authenticate via iam:

```shell
dokku mariadb:backup-schedule lolipop "0 3 * * *" my-s3-bucket --use-iam
```
### cat the contents of the configured backup cronfile for the service

```shell
# usage
dokku mariadb:backup-schedule-cat <service>
```

examples:

Cat the contents of the configured backup cronfile for the service:

```shell
dokku mariadb:backup-schedule-cat lolipop
```
### unschedules the backup of the mariadb service

```shell
# usage
dokku mariadb:backup-unschedule <service>
```

examples:

Remove the scheduled backup from cron:

```shell
dokku mariadb:backup-unschedule lolipop
```

### Disabling `docker pull` calls

If you wish to disable the `docker pull` calls that the plugin triggers, you may set the `MARIADB_DISABLE_PULL` environment variable to `true`. Once disabled, you will need to pull the service image you wish to deploy as shown in the `stderr` output.

Please ensure the proper images are in place when `docker pull` is disabled.