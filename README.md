# dokku mariadb [![Build Status](https://img.shields.io/github/actions/workflow/status/dokku/dokku-mariadb/ci.yml?branch=master&style=flat-square "Build Status")](https://github.com/dokku/dokku-mariadb/actions/workflows/ci.yml?query=branch%3Amaster) [![IRC Network](https://img.shields.io/badge/irc-libera-blue.svg?style=flat-square "IRC Libera")](https://webchat.libera.chat/?channels=dokku)

Official mariadb plugin for dokku. Currently defaults to installing [mariadb 11.0.3](https://hub.docker.com/_/mariadb/).

## Requirements

- dokku 0.19.x+
- docker 1.8.x

## Installation

```shell
# on 0.19.x+
sudo dokku plugin:install https://github.com/dokku/dokku-mariadb.git mariadb
```

## Commands

```
mariadb:app-links <app>                            # list all mariadb service links for a given app
mariadb:backup <service> <bucket-name> [--use-iam] # create a backup of the mariadb service to an existing s3 bucket
mariadb:backup-auth <service> <aws-access-key-id> <aws-secret-access-key> <aws-default-region> <aws-signature-version> <endpoint-url> # set up authentication for backups on the mariadb service
mariadb:backup-deauth <service>                    # remove backup authentication for the mariadb service
mariadb:backup-schedule <service> <schedule> <bucket-name> [--use-iam] # schedule a backup of the mariadb service
mariadb:backup-schedule-cat <service>              # cat the contents of the configured backup cronfile for the service
mariadb:backup-set-encryption <service> <passphrase> # set encryption for all future backups of mariadb service
mariadb:backup-unschedule <service>                # unschedule the backup of the mariadb service
mariadb:backup-unset-encryption <service>          # unset encryption for future backups of the mariadb service
mariadb:clone <service> <new-service> [--clone-flags...] # create container <new-name> then copy data from <name> into <new-name>
mariadb:connect <service>                          # connect to the service via the mariadb connection tool
mariadb:create <service> [--create-flags...]       # create a mariadb service
mariadb:destroy <service> [-f|--force]             # delete the mariadb service/data/container if there are no links left
mariadb:enter <service>                            # enter or run a command in a running mariadb service container
mariadb:exists <service>                           # check if the mariadb service exists
mariadb:export <service>                           # export a dump of the mariadb service database
mariadb:expose <service> <ports...>                # expose a mariadb service on custom host:port if provided (random port on the 0.0.0.0 interface if otherwise unspecified)
mariadb:import <service>                           # import a dump into the mariadb service database
mariadb:info <service> [--single-info-flag]        # print the service information
mariadb:link <service> <app> [--link-flags...]     # link the mariadb service to the app
mariadb:linked <service> <app>                     # check if the mariadb service is linked to an app
mariadb:links <service>                            # list all apps linked to the mariadb service
mariadb:list                                       # list all mariadb services
mariadb:logs <service> [-t|--tail] <tail-num-optional> # print the most recent log(s) for this service
mariadb:pause <service>                            # pause a running mariadb service
mariadb:promote <service> <app>                    # promote service <service> as DATABASE_URL in <app>
mariadb:restart <service>                          # graceful shutdown and restart of the mariadb service container
mariadb:set <service> <key> <value>                # set or clear a property for a service
mariadb:start <service>                            # start a previously stopped mariadb service
mariadb:stop <service>                             # stop a running mariadb service
mariadb:unexpose <service>                         # unexpose a previously exposed mariadb service
mariadb:unlink <service> <app>                     # unlink the mariadb service from the app
mariadb:upgrade <service> [--upgrade-flags...]     # upgrade service <service> to the specified versions
```

## Usage

Help for any commands can be displayed by specifying the command as an argument to mariadb:help. Plugin help output in conjunction with any files in the `docs/` folder is used to generate the plugin documentation. Please consult the `mariadb:help` command for any undocumented commands.

### Basic Usage

### create a mariadb service

```shell
# usage
dokku mariadb:create <service> [--create-flags...]
```

flags:

- `-c|--config-options "--args --go=here"`: extra arguments to pass to the container create command (default: `None`)
- `-C|--custom-env "USER=alpha;HOST=beta"`: semi-colon delimited environment variables to start the service with
- `-i|--image IMAGE`: the image name to start the service with
- `-I|--image-version IMAGE_VERSION`: the image version to start the service with
- `-m|--memory MEMORY`: container memory limit in megabytes (default: unlimited)
- `-N|--initial-network INITIAL_NETWORK`: the initial network to attach the service to
- `-p|--password PASSWORD`: override the user-level service password
- `-P|--post-create-network NETWORKS`: a comman-separated list of networks to attach the service container to after service creation
- `-r|--root-password PASSWORD`: override the root-level service password
- `-S|--post-start-network NETWORKS`: a comman-separated list of networks to attach the service container to after service start
- `-s|--shm-size SHM_SIZE`: override shared memory size for mariadb docker container

Create a mariadb service named lollipop:

```shell
dokku mariadb:create lollipop
```

You can also specify the image and image version to use for the service. It *must* be compatible with the mariadb image.

```shell
export MARIADB_IMAGE="mariadb"
export MARIADB_IMAGE_VERSION="${PLUGIN_IMAGE_VERSION}"
dokku mariadb:create lollipop
```

You can also specify custom environment variables to start the mariadb service in semi-colon separated form.

```shell
export MARIADB_CUSTOM_ENV="USER=alpha;HOST=beta"
dokku mariadb:create lollipop
```

### print the service information

```shell
# usage
dokku mariadb:info <service> [--single-info-flag]
```

flags:

- `--config-dir`: show the service configuration directory
- `--data-dir`: show the service data directory
- `--dsn`: show the service DSN
- `--exposed-ports`: show service exposed ports
- `--id`: show the service container id
- `--internal-ip`: show the service internal ip
- `--initial-network`: show the initial network being connected to
- `--links`: show the service app links
- `--post-create-network`: show the networks to attach to after service container creation
- `--post-start-network`: show the networks to attach to after service container start
- `--service-root`: show the service root directory
- `--status`: show the service running status
- `--version`: show the service image version

Get connection information as follows:

```shell
dokku mariadb:info lollipop
```

You can also retrieve a specific piece of service info via flags:

```shell
dokku mariadb:info lollipop --config-dir
dokku mariadb:info lollipop --data-dir
dokku mariadb:info lollipop --dsn
dokku mariadb:info lollipop --exposed-ports
dokku mariadb:info lollipop --id
dokku mariadb:info lollipop --internal-ip
dokku mariadb:info lollipop --initial-network
dokku mariadb:info lollipop --links
dokku mariadb:info lollipop --post-create-network
dokku mariadb:info lollipop --post-start-network
dokku mariadb:info lollipop --service-root
dokku mariadb:info lollipop --status
dokku mariadb:info lollipop --version
```

### list all mariadb services

```shell
# usage
dokku mariadb:list 
```

List all services:

```shell
dokku mariadb:list
```

### print the most recent log(s) for this service

```shell
# usage
dokku mariadb:logs <service> [-t|--tail] <tail-num-optional>
```

flags:

- `-t|--tail [<tail-num>]`: do not stop when end of the logs are reached and wait for additional output

You can tail logs for a particular service:

```shell
dokku mariadb:logs lollipop
```

By default, logs will not be tailed, but you can do this with the --tail flag:

```shell
dokku mariadb:logs lollipop --tail
```

The default tail setting is to show all logs, but an initial count can also be specified:

```shell
dokku mariadb:logs lollipop --tail 5
```

### link the mariadb service to the app

```shell
# usage
dokku mariadb:link <service> <app> [--link-flags...]
```

flags:

- `-a|--alias "BLUE_DATABASE"`: an alternative alias to use for linking to an app via environment variable
- `-q|--querystring "pool=5"`: ampersand delimited querystring arguments to append to the service link
- `-n|--no-restart "false"`: whether or not to restart the app on link (default: true)

A mariadb service can be linked to a container. This will use native docker links via the docker-options plugin. Here we link it to our `playground` app.

> NOTE: this will restart your app

```shell
dokku mariadb:link lollipop playground
```

The following environment variables will be set automatically by docker (not on the app itself, so they won’t be listed when calling dokku config):

```
DOKKU_MARIADB_LOLLIPOP_NAME=/lollipop/DATABASE
DOKKU_MARIADB_LOLLIPOP_PORT=tcp://172.17.0.1:3306
DOKKU_MARIADB_LOLLIPOP_PORT_3306_TCP=tcp://172.17.0.1:3306
DOKKU_MARIADB_LOLLIPOP_PORT_3306_TCP_PROTO=tcp
DOKKU_MARIADB_LOLLIPOP_PORT_3306_TCP_PORT=3306
DOKKU_MARIADB_LOLLIPOP_PORT_3306_TCP_ADDR=172.17.0.1
```

The following will be set on the linked application by default:

```
DATABASE_URL=mysql://mariadb:SOME_PASSWORD@dokku-mariadb-lollipop:3306/lollipop
```

The host exposed here only works internally in docker containers. If you want your container to be reachable from outside, you should use the `expose` subcommand. Another service can be linked to your app:

```shell
dokku mariadb:link other_service playground
```

It is possible to change the protocol for `DATABASE_URL` by setting the environment variable `MARIADB_DATABASE_SCHEME` on the app. Doing so will after linking will cause the plugin to think the service is not linked, and we advise you to unlink before proceeding.

```shell
dokku config:set playground MARIADB_DATABASE_SCHEME=mysql2
dokku mariadb:link lollipop playground
```

This will cause `DATABASE_URL` to be set as:

```
mysql2://mariadb:SOME_PASSWORD@dokku-mariadb-lollipop:3306/lollipop
```

### unlink the mariadb service from the app

```shell
# usage
dokku mariadb:unlink <service> <app>
```

flags:

- `-n|--no-restart "false"`: whether or not to restart the app on unlink (default: true)

You can unlink a mariadb service:

> NOTE: this will restart your app and unset related environment variables

```shell
dokku mariadb:unlink lollipop playground
```

### set or clear a property for a service

```shell
# usage
dokku mariadb:set <service> <key> <value>
```

Set the network to attach after the service container is started:

```shell
dokku mariadb:set lollipop post-create-network custom-network
```

Set multiple networks:

```shell
dokku mariadb:set lollipop post-create-network custom-network,other-network
```

Unset the post-create-network value:

```shell
dokku mariadb:set lollipop post-create-network
```

### Service Lifecycle

The lifecycle of each service can be managed through the following commands:

### connect to the service via the mariadb connection tool

```shell
# usage
dokku mariadb:connect <service>
```

Connect to the service via the mariadb connection tool:

> NOTE: disconnecting from ssh while running this command may leave zombie processes due to moby/moby#9098

```shell
dokku mariadb:connect lollipop
```

### enter or run a command in a running mariadb service container

```shell
# usage
dokku mariadb:enter <service>
```

A bash prompt can be opened against a running service. Filesystem changes will not be saved to disk.

> NOTE: disconnecting from ssh while running this command may leave zombie processes due to moby/moby#9098

```shell
dokku mariadb:enter lollipop
```

You may also run a command directly against the service. Filesystem changes will not be saved to disk.

```shell
dokku mariadb:enter lollipop touch /tmp/test
```

### expose a mariadb service on custom host:port if provided (random port on the 0.0.0.0 interface if otherwise unspecified)

```shell
# usage
dokku mariadb:expose <service> <ports...>
```

Expose the service on the service's normal ports, allowing access to it from the public interface (`0.0.0.0`):

```shell
dokku mariadb:expose lollipop 3306
```

Expose the service on the service's normal ports, with the first on a specified ip adddress (127.0.0.1):

```shell
dokku mariadb:expose lollipop 127.0.0.1:3306
```

### unexpose a previously exposed mariadb service

```shell
# usage
dokku mariadb:unexpose <service>
```

Unexpose the service, removing access to it from the public interface (`0.0.0.0`):

```shell
dokku mariadb:unexpose lollipop
```

### promote service <service> as DATABASE_URL in <app>

```shell
# usage
dokku mariadb:promote <service> <app>
```

If you have a mariadb service linked to an app and try to link another mariadb service another link environment variable will be generated automatically:

```
DOKKU_DATABASE_BLUE_URL=mysql://other_service:ANOTHER_PASSWORD@dokku-mariadb-other-service:3306/other_service
```

You can promote the new service to be the primary one:

> NOTE: this will restart your app

```shell
dokku mariadb:promote other_service playground
```

This will replace `DATABASE_URL` with the url from other_service and generate another environment variable to hold the previous value if necessary. You could end up with the following for example:

```
DATABASE_URL=mysql://other_service:ANOTHER_PASSWORD@dokku-mariadb-other-service:3306/other_service
DOKKU_DATABASE_BLUE_URL=mysql://other_service:ANOTHER_PASSWORD@dokku-mariadb-other-service:3306/other_service
DOKKU_DATABASE_SILVER_URL=mysql://lollipop:SOME_PASSWORD@dokku-mariadb-lollipop:3306/lollipop
```

### start a previously stopped mariadb service

```shell
# usage
dokku mariadb:start <service>
```

Start the service:

```shell
dokku mariadb:start lollipop
```

### stop a running mariadb service

```shell
# usage
dokku mariadb:stop <service>
```

Stop the service and removes the running container:

```shell
dokku mariadb:stop lollipop
```

### pause a running mariadb service

```shell
# usage
dokku mariadb:pause <service>
```

Pause the running container for the service:

```shell
dokku mariadb:pause lollipop
```

### graceful shutdown and restart of the mariadb service container

```shell
# usage
dokku mariadb:restart <service>
```

Restart the service:

```shell
dokku mariadb:restart lollipop
```

### upgrade service <service> to the specified versions

```shell
# usage
dokku mariadb:upgrade <service> [--upgrade-flags...]
```

flags:

- `-c|--config-options "--args --go=here"`: extra arguments to pass to the container create command (default: `None`)
- `-C|--custom-env "USER=alpha;HOST=beta"`: semi-colon delimited environment variables to start the service with
- `-i|--image IMAGE`: the image name to start the service with
- `-I|--image-version IMAGE_VERSION`: the image version to start the service with
- `-N|--initial-network INITIAL_NETWORK`: the initial network to attach the service to
- `-P|--post-create-network NETWORKS`: a comman-separated list of networks to attach the service container to after service creation
- `-R|--restart-apps "true"`: whether or not to force an app restart (default: false)
- `-S|--post-start-network NETWORKS`: a comman-separated list of networks to attach the service container to after service start
- `-s|--shm-size SHM_SIZE`: override shared memory size for mariadb docker container

You can upgrade an existing service to a new image or image-version:

```shell
dokku mariadb:upgrade lollipop
```

### Service Automation

Service scripting can be executed using the following commands:

### list all mariadb service links for a given app

```shell
# usage
dokku mariadb:app-links <app>
```

List all mariadb services that are linked to the `playground` app.

```shell
dokku mariadb:app-links playground
```

### create container <new-name> then copy data from <name> into <new-name>

```shell
# usage
dokku mariadb:clone <service> <new-service> [--clone-flags...]
```

flags:

- `-c|--config-options "--args --go=here"`: extra arguments to pass to the container create command (default: `None`)
- `-C|--custom-env "USER=alpha;HOST=beta"`: semi-colon delimited environment variables to start the service with
- `-i|--image IMAGE`: the image name to start the service with
- `-I|--image-version IMAGE_VERSION`: the image version to start the service with
- `-m|--memory MEMORY`: container memory limit in megabytes (default: unlimited)
- `-N|--initial-network INITIAL_NETWORK`: the initial network to attach the service to
- `-p|--password PASSWORD`: override the user-level service password
- `-P|--post-create-network NETWORKS`: a comman-separated list of networks to attach the service container to after service creation
- `-r|--root-password PASSWORD`: override the root-level service password
- `-S|--post-start-network NETWORKS`: a comman-separated list of networks to attach the service container to after service start
- `-s|--shm-size SHM_SIZE`: override shared memory size for mariadb docker container

You can clone an existing service to a new one:

```shell
dokku mariadb:clone lollipop lollipop-2
```

### check if the mariadb service exists

```shell
# usage
dokku mariadb:exists <service>
```

Here we check if the lollipop mariadb service exists.

```shell
dokku mariadb:exists lollipop
```

### check if the mariadb service is linked to an app

```shell
# usage
dokku mariadb:linked <service> <app>
```

Here we check if the lollipop mariadb service is linked to the `playground` app.

```shell
dokku mariadb:linked lollipop playground
```

### list all apps linked to the mariadb service

```shell
# usage
dokku mariadb:links <service>
```

List all apps linked to the `lollipop` mariadb service.

```shell
dokku mariadb:links lollipop
```

### Data Management

The underlying service data can be imported and exported with the following commands:

### import a dump into the mariadb service database

```shell
# usage
dokku mariadb:import <service>
```

Import a datastore dump:

```shell
dokku mariadb:import lollipop < data.dump
```

### export a dump of the mariadb service database

```shell
# usage
dokku mariadb:export <service>
```

By default, datastore output is exported to stdout:

```shell
dokku mariadb:export lollipop
```

You can redirect this output to a file:

```shell
dokku mariadb:export lollipop > data.dump
```

### Backups

Datastore backups are supported via AWS S3 and S3 compatible services like [minio](https://github.com/minio/minio).

You may skip the `backup-auth` step if your dokku install is running within EC2 and has access to the bucket via an IAM profile. In that case, use the `--use-iam` option with the `backup` command.

Backups can be performed using the backup commands:

### set up authentication for backups on the mariadb service

```shell
# usage
dokku mariadb:backup-auth <service> <aws-access-key-id> <aws-secret-access-key> <aws-default-region> <aws-signature-version> <endpoint-url>
```

Setup s3 backup authentication:

```shell
dokku mariadb:backup-auth lollipop AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY
```

Setup s3 backup authentication with different region:

```shell
dokku mariadb:backup-auth lollipop AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_REGION
```

Setup s3 backup authentication with different signature version and endpoint:

```shell
dokku mariadb:backup-auth lollipop AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_REGION AWS_SIGNATURE_VERSION ENDPOINT_URL
```

More specific example for minio auth:

```shell
dokku mariadb:backup-auth lollipop MINIO_ACCESS_KEY_ID MINIO_SECRET_ACCESS_KEY us-east-1 s3v4 https://YOURMINIOSERVICE
```

### remove backup authentication for the mariadb service

```shell
# usage
dokku mariadb:backup-deauth <service>
```

Remove s3 authentication:

```shell
dokku mariadb:backup-deauth lollipop
```

### create a backup of the mariadb service to an existing s3 bucket

```shell
# usage
dokku mariadb:backup <service> <bucket-name> [--use-iam]
```

flags:

- `-u|--use-iam`: use the IAM profile associated with the current server

Backup the `lollipop` service to the `my-s3-bucket` bucket on `AWS`:`

```shell
dokku mariadb:backup lollipop my-s3-bucket --use-iam
```

Restore a backup file (assuming it was extracted via `tar -xf backup.tgz`):

```shell
dokku mariadb:import lollipop < backup-folder/export
```

### set encryption for all future backups of mariadb service

```shell
# usage
dokku mariadb:backup-set-encryption <service> <passphrase>
```

Set the GPG-compatible passphrase for encrypting backups for backups:

```shell
dokku mariadb:backup-set-encryption lollipop
```

### unset encryption for future backups of the mariadb service

```shell
# usage
dokku mariadb:backup-unset-encryption <service>
```

Unset the `GPG` encryption passphrase for backups:

```shell
dokku mariadb:backup-unset-encryption lollipop
```

### schedule a backup of the mariadb service

```shell
# usage
dokku mariadb:backup-schedule <service> <schedule> <bucket-name> [--use-iam]
```

flags:

- `-u|--use-iam`: use the IAM profile associated with the current server

Schedule a backup:

> 'schedule' is a crontab expression, eg. "0 3 * * *" for each day at 3am

```shell
dokku mariadb:backup-schedule lollipop "0 3 * * *" my-s3-bucket
```

Schedule a backup and authenticate via iam:

```shell
dokku mariadb:backup-schedule lollipop "0 3 * * *" my-s3-bucket --use-iam
```

### cat the contents of the configured backup cronfile for the service

```shell
# usage
dokku mariadb:backup-schedule-cat <service>
```

Cat the contents of the configured backup cronfile for the service:

```shell
dokku mariadb:backup-schedule-cat lollipop
```

### unschedule the backup of the mariadb service

```shell
# usage
dokku mariadb:backup-unschedule <service>
```

Remove the scheduled backup from cron:

```shell
dokku mariadb:backup-unschedule lollipop
```

### Disabling `docker image pull` calls

If you wish to disable the `docker image pull` calls that the plugin triggers, you may set the `MARIADB_DISABLE_PULL` environment variable to `true`. Once disabled, you will need to pull the service image you wish to deploy as shown in the `stderr` output.

Please ensure the proper images are in place when `docker image pull` is disabled.
