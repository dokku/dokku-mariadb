# dokku mariadb [![Build Status](https://img.shields.io/github/actions/workflow/status/dokku/dokku-mariadb/ci.yml?branch=master&style=flat-square "Build Status")](https://github.com/dokku/dokku-mariadb/actions/workflows/ci.yml?query=branch%3Amaster) [![IRC Network](https://img.shields.io/badge/irc-libera-blue.svg?style=flat-square "IRC Libera")](https://webchat.libera.chat/?channels=dokku)

Official mariadb plugin for dokku. Currently defaults to installing [mariadb 13.0.2](https://hub.docker.com/_/mariadb/).

## Requirements

- dokku 0.35.x+
- docker 1.8.x

## Installation

```shell
# on 0.35.x+
sudo dokku plugin:install https://github.com/dokku/dokku-mariadb.git --name mariadb
```

## Commands

```
mariadb:app-links [<app>]                          # list all MariaDB service links for a given app
mariadb:backup <service> <bucket-name> [-u|--use-iam] # create a backup of the MariaDB service to an existing s3 bucket
mariadb:backup-auth <service> <aws-access-key-id> <aws-secret-access-key> <aws-default-region> <aws-signature-version> <endpoint-url> # set up authentication for backups on the MariaDB service
mariadb:backup-deauth <service>                    # remove backup authentication for the MariaDB service
mariadb:backup-schedule <service> <schedule> <bucket-name> [-u|--use-iam] # schedule a backup of the MariaDB service
mariadb:backup-schedule-cat <service>              # cat the crontab line of the scheduled backup for the service
mariadb:backup-set-encryption <service> <passphrase> # set encryption for all future backups of MariaDB service
mariadb:backup-set-public-key-encryption <service> <public-key-id> # set GPG Public Key encryption for all future backups of MariaDB service
mariadb:backup-unschedule <service>                # unschedule the backup of the MariaDB service
mariadb:backup-unset-encryption <service>          # unset encryption for future backups of the MariaDB service
mariadb:backup-unset-public-key-encryption <service> # unset GPG Public Key encryption for future backups of the MariaDB service
mariadb:clone <service> <new-service> [--clone-flags...] # create container <new-name> then copy data from <name> into <new-name>
mariadb:connect <service>                          # connect to the service via the mariadb connection tool
mariadb:create <service> [--create-flags...]       # create a MariaDB service
mariadb:destroy <service> [-f|--force]             # delete the MariaDB service/data/container if there are no links left
mariadb:enter <service>                            # enter or run a command in a running MariaDB service container
mariadb:exists <service>                           # check if the MariaDB service exists
mariadb:export <service> [-f|--file <path>] [--force] [-- <export-args...>] # export a dump of the MariaDB service database
mariadb:expose <service> <ports...>                # expose a MariaDB service on custom host:port if provided (random port on the 0.0.0.0 interface if otherwise unspecified)
mariadb:import <service> [-f|--file <path>] [-- <import-args...>] # import a dump into the MariaDB service database
mariadb:info [<service>] [--info-flags...]         # print the service information
mariadb:link <service> [<app>] [--link-flags...]   # link the MariaDB service to the app
mariadb:linked <service> [<app>]                   # check if the MariaDB service is linked to an app
mariadb:links <service>                            # list all apps linked to the MariaDB service
mariadb:list                                       # list all MariaDB services
mariadb:logs <service> [-t|--tail [<tail-num>]]    # print the most recent log(s) for this service
mariadb:mount [--replace] <service> <source:container-dir[:options]>... # mount a host path or docker volume into the service container
mariadb:pause <service>                            # pause a running MariaDB service
mariadb:promote <service> [<app>]                  # promote service <service> as DATABASE_URL in <app>
mariadb:reexpose <service>                         # reexpose a MariaDB service, applying its expose settings without restarting it
mariadb:restart <service>                          # graceful shutdown and restart of the MariaDB service container
mariadb:set <service> <key> <value>                # set or clear a property for a service
mariadb:start <service>                            # start a previously stopped MariaDB service
mariadb:stop <service>                             # stop a running MariaDB service
mariadb:unexpose <service>                         # unexpose a previously exposed MariaDB service
mariadb:unlink <service> [<app>] [-n|--no-restart] # unlink the MariaDB service from the app
mariadb:unmount [--all] <service> [<source:container-dir>...] # remove one or all mounts from the service container
mariadb:upgrade <service> [--upgrade-flags...]     # upgrade service <service> to the specified versions
```

## Usage

Help for any commands can be displayed by specifying the command as an argument to mariadb:help. Plugin help output in conjunction with any files in the `docs/` folder is used to generate the plugin documentation. Please consult the `mariadb:help` command for any undocumented commands.

### Basic Usage

### create a MariaDB service

```shell
# usage
dokku mariadb:create <service> [--create-flags...]
```

flags:

- `-c|--config-options <string>`: extra arguments for the process the service container runs, not docker flags; use mount for mounts
- `-C|--custom-env <string>`: semi-colon delimited environment variables to start the service with
- `-i|--image <string>`: the image name to start the service with
- `-I|--image-version <string>`: the image version to start the service with
- `-N|--initial-network <string>`: the initial network to attach the service to
- `--log-driver <string>`: the docker logging driver to run the service container with (default: the daemon's own)
- `--log-opt <strings>`: a comma-separated list of key=value docker log options for the service container
- `-m|--memory <int>`: container memory limit in megabytes (default: unlimited)
- `-p|--password <string>`: override the user-level service password, for datastores that have one
- `-P|--post-create-network <strings>`: a comma-separated list of networks to attach the service container to after service creation
- `-S|--post-start-network <strings>`: a comma-separated list of networks to attach the service container to after service start
- `--restart <string>`: the docker restart policy to run the service container with (default: always)
- `-r|--root-password <string>`: override the root-level service password, for datastores that have one
- `-s|--shm-size <string>`: override shared memory size for the service docker container
- `--volume <stringArray>`: a host path or docker volume to mount into the service container, as <source>:<container-dir>[:<options>], repeatable
- `--wait-timeout <string>`: seconds to wait for the service to become ready (default: the datastore's own)

Create a mariadb service named lollipop:

```shell
dokku mariadb:create lollipop
```

You can also specify the image and image version to use for the service. It *must* be compatible with the mariadb image.

```shell
export MARIADB_IMAGE="mariadb"
export MARIADB_IMAGE_VERSION="13.0.2"
dokku mariadb:create lollipop
```

An image other than mariadb has no version to fall back on, because the version this plugin pins belongs to mariadb, so name one alongside it.

```shell
dokku mariadb:create lollipop --image <image> --image-version <version>
```

You can also specify custom environment variables to start the mariadb service in semicolon-separated form.

```shell
export MARIADB_CUSTOM_ENV="USER=alpha;HOST=beta"
dokku mariadb:create lollipop
```

The container log is bounded by whatever `dokku logs:set --global max-size` says, and by dokku's own default where it says nothing, which a service may override for itself.

```shell
dokku mariadb:create lollipop --log-opt max-size=20m,max-file=3
```

The container is restarted by docker whenever it stops, which a service may change for itself.

```shell
dokku mariadb:create lollipop --restart unless-stopped
```

The service is waited on until it answers, for as long as the datastore's own default, which a slow host may raise for every service with `MARIADB_WAIT_TIMEOUT` or a service may raise for itself.

```shell
dokku mariadb:create lollipop --wait-timeout 120
```

The config options are handed to the process the container runs, not to docker, so a host path or docker volume is mounted with --volume, which may be repeated.

```shell
dokku mariadb:create lollipop --volume /var/lib/dokku/data/storage/lollipop:/opt/extra:ro
```

The service passwords are generated unless they are given. A datastore without a root password refuses --root-password rather than dropping it.

```shell
dokku mariadb:create lollipop --password <password> --root-password <root-password>
```

### delete the MariaDB service/data/container if there are no links left

```shell
# usage
dokku mariadb:destroy <service> [-f|--force]
```

flags:

- `-f|--force`: force the destruction of the service

Destroy the service, it's data, and the running container:

```shell
dokku mariadb:destroy lollipop
```

A service that is still linked to an app is not destroyed, and the apps it is linked to are named. Unlink them first.

### print the service information

```shell
# usage
dokku mariadb:info [<service>] [--info-flags...]
```

flags:

- `--backend`: show the execution backend the service was created with
- `--backup-auth-fingerprint`: show a sha256 fingerprint of the stored backup access key id and secret
- `--backup-authenticated`: show whether backup credentials are stored for the service
- `--backup-bucket`: show the bucket scheduled backups are shipped to
- `--backup-default-region`: show the region backups authenticate against
- `--backup-encrypted`: show whether scheduled backups are encrypted with a passphrase
- `--backup-encryption-fingerprint`: show a sha256 fingerprint of the stored backup passphrase
- `--backup-endpoint-url`: show the s3-compatible endpoint backups are shipped to
- `--backup-keyserver`: show the keyserver backup public keys are fetched from
- `--backup-public-key-id`: show the gpg public key id backups are encrypted with
- `--backup-schedule`: show the cron schedule backups run on
- `--backup-signature-version`: show the signature version backups authenticate with
- `--backup-use-iam`: show whether scheduled backups authenticate with an instance role
- `--config-dir`: show the service configuration directory
- `--config-options`: show the config options the service container is run with
- `--custom-env`: show the custom environment the service container is run with
- `--data-dir`: show the service data directory
- `--database-name`: show the name of the database inside the service
- `--definition`: show the definition the service was created with
- `--dsn`: show the service DSN
- `--export-args`: show the extra arguments every export of the service is run with
- `--expose-address`: show the address exposed ports without one of their own are published on
- `--expose-source-range`: show the only range of client addresses the exposed ports accept
- `--exposed-ports`: show service exposed ports
- `--id`: show the service container id
- `--image`: show the image the service runs
- `--image-version`: show the image version the service was created with
- `--import-args`: show the extra arguments every import into the service is run with
- `--initial-network`: show the initial network being connected to
- `--internal-ip`: show the service internal ip
- `--links`: show the service app links
- `--log-driver`: show the docker logging driver the service container is run with
- `--log-opt`: show the docker log options the service container is run with
- `--memory`: show the memory limit the service container is run with
- `--mounts`: show the host paths and docker volumes mounted into the service container
- `--post-create-network`: show the networks to attach to after service container creation
- `--post-start-network`: show the networks to attach to after service container start
- `--restart-policy`: show the restart policy the service container is run with
- `--service`: show the name of the service
- `--service-root`: show the service root directory
- `--shm-size`: show the shared memory size the service container is run with
- `--status`: show the service running status
- `--version`: show the service image version
- `--wait-timeout`: show the seconds the service is waited on to become ready

Get connection information as follows:

```shell
dokku mariadb:info lollipop
```

Alongside the connection information this reports the properties set on the service, the state it was created with, and its backup settings. A property that was never set, or that was unset, reports as empty. Omit the service to report on every mariadb service:

```shell
dokku mariadb:info
```

The information can be read by machine, one json object per service:

```shell
dokku mariadb:info lollipop --format json
```

You can also retrieve a specific piece of service info via a flag, which prints it on its own:

```shell
dokku mariadb:info lollipop --dsn
dokku mariadb:info lollipop --status
dokku mariadb:info lollipop --initial-network
```

> NOTE: a flag cannot be combined with --format, and only one may be given

The properties mariadb:set writes are reported under the names it takes, so a value read here can be written back:

```shell
dokku mariadb:set lollipop initial-network my-network
```

The stored backup credentials and passphrase are never printed. Each is reported as a lowercase hex sha256 fingerprint of the stored value, with surrounding whitespace trimmed, so a copy of the values can be compared against it:

```shell
dokku mariadb:info lollipop --backup-auth-fingerprint
dokku mariadb:info lollipop --backup-encryption-fingerprint
```

The same fingerprints can be computed from the values that were passed to backup-auth and backup-set-encryption:

```
printf '%s\n%s' "$AWS_ACCESS_KEY_ID" "$AWS_SECRET_ACCESS_KEY" | sha256sum
printf '%s' "$PASSPHRASE" | sha256sum
```

### list all MariaDB services

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
dokku mariadb:logs <service> [-t|--tail [<tail-num>]]
```

flags:

- `-t|--tail <int>`: tail the logs, optionally showing this many lines

You can tail logs for a particular service:

```shell
dokku mariadb:logs lollipop
```

By default, logs will not be tailed, but you can do this with the --tail flag:

```shell
dokku mariadb:logs lollipop --tail
```

By default the last 100 lines are shown, but a different count can be specified:

```shell
dokku mariadb:logs lollipop --tail=5
```

### link the MariaDB service to the app

```shell
# usage
dokku mariadb:link <service> [<app>] [--link-flags...]
```

flags:

- `-a|--alias <string>`: the prefix of the config variable the service url is set as on the app, which is suffixed with _URL
- `-n|--no-restart`: whether to skip restarting the app
- `-q|--querystring <string>`: ampersand delimited querystring arguments to append to the service url after a ?

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
DATABASE_URL=mysql://:SOME_PASSWORD@dokku-mariadb-lollipop:3306
```

The host exposed here only works internally in docker containers. If you want your container to be reachable from outside, you should use the `expose` subcommand. Another service can be linked to your app:

```shell
dokku mariadb:link other_service playground
```

The url can be set under another name with the `--alias` flag. The value given is the prefix of the config variable, which is suffixed with `_URL` and holds the same url:

```shell
dokku mariadb:link lollipop playground --alias BLUE_DATABASE
```

This will set the following on the linked application instead of `DATABASE_URL`:

```
BLUE_DATABASE_URL=mysql://:SOME_PASSWORD@dokku-mariadb-lollipop:3306
```

An alias whose variable is already set on the app is refused, and unlink removes the variable whatever alias it was set under. Arguments can be appended to the url as a querystring with the `--querystring` flag:

```shell
dokku mariadb:link lollipop playground --querystring "foo=bar&baz=qux"
```

This will cause `DATABASE_URL` to be set as:

```
mysql://:SOME_PASSWORD@dokku-mariadb-lollipop:3306?foo=bar&baz=qux
```

It is possible to change the protocol for `DATABASE_URL` by setting the environment variable `MARIADB_DATABASE_SCHEME` on the app. Doing so after linking means unlink no longer finds the variable it set, and leaves it in place, so we advise you to unlink before proceeding.

```shell
dokku config:set playground MARIADB_DATABASE_SCHEME=mysql2
dokku mariadb:link lollipop playground
```

This will cause `DATABASE_URL` to be set as:

```
mysql2://:SOME_PASSWORD@dokku-mariadb-lollipop:3306
```

### unlink the MariaDB service from the app

```shell
# usage
dokku mariadb:unlink <service> [<app>] [-n|--no-restart]
```

flags:

- `-n|--no-restart`: whether to skip restarting the app

You can unlink a mariadb service:

> NOTE: this will restart your app and unset related environment variables

```shell
dokku mariadb:unlink lollipop playground
```

An app is still linked after its `DATABASE_URL` is changed to point elsewhere, and is unlinked the same way. The variable it now holds is not the service's, so it is left alone, nothing is unset, the app is not restarted, and a warning says so.

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

Set the keyserver a public key for backup encryption is fetched from:

```shell
dokku mariadb:set lollipop backup-keyserver hkp://keys.example.com
```

Cap the container log at a size of your own rather than the one it inherits:

```shell
dokku mariadb:set lollipop log-opt max-size=20m,max-file=3
```

Keep the log unbounded, which is what a service had before there was anything to say here:

```shell
dokku mariadb:set lollipop log-opt max-size=unlimited
```

Send the container log somewhere other than the daemon's own driver:

```shell
dokku mariadb:set lollipop log-driver journald
```

Restart the container unless it was stopped on purpose, including across a docker restart:

```shell
dokku mariadb:set lollipop restart-policy unless-stopped
```

Go back to always restarting the container:

```shell
dokku mariadb:set lollipop restart-policy
```

Wait up to two minutes for the service to answer, used the next time it is started:

```shell
dokku mariadb:set lollipop wait-timeout 120
```

Go back to the wait timeout the host or the datastore sets:

```shell
dokku mariadb:set lollipop wait-timeout
```

Publish exposed ports that have no address of their own on one address rather than on every interface:

```shell
dokku mariadb:set lollipop expose-address 10.0.0.5
```

Only accept connections to the exposed ports from clients in one `IP` address or `CIDR`:

```shell
dokku mariadb:set lollipop expose-source-range 10.0.0.0/8
```

Go back to accepting every client:

```shell
dokku mariadb:set lollipop expose-source-range
```

Pass extra arguments to every export of the service, including the ones backups and clones make. The value follows -- so that its leading dash is not read as a flag, and an argument with a space in it is quoted:

```shell
dokku mariadb:set lollipop export-args -- "<export-args...>"
```

Go back to exporting with the datastore's own arguments alone:

```shell
dokku mariadb:set lollipop export-args
```

Pass extra arguments to every import into the service, including the one a clone makes:

```shell
dokku mariadb:set lollipop import-args -- "<import-args...>"
```

Go back to importing with the datastore's own arguments alone:

```shell
dokku mariadb:set lollipop import-args
```

> NOTE: a log setting or a restart policy reaches the container the next time one is built. mariadb:restart keeps the container it has, so use mariadb:stop and then mariadb:start on a service that is already running.
> NOTE: an expose-address or expose-source-range reaches an exposed service with mariadb:reexpose, which replaces the container publishing its ports and leaves the service container running.

### mount a host path or docker volume into the service container

```shell
# usage
dokku mariadb:mount [--replace] <service> <source:container-dir[:options]>...
```

flags:

- `--replace`: replace the service's entire set of mounts with the ones given
- `--volume-chown <string>`: who to hand the mounted directory to, for a host path inside the service's directory; not valid with --replace
- `--volume-options <string>`: comma-separated docker mount options, such as z or nocopy; not valid with --replace
- `--volume-readonly`: mount the volume read only; not valid with --replace
- `--volume-subpath <string>`: a subpath within the source to mount rather than the source itself; not valid with --replace

Mount a host directory into the service container:

```shell
dokku mariadb:mount lollipop /var/lib/dokku/data/storage/lollipop:/opt/extra
```

The source is an absolute host path, which must already exist, or the name of a docker volume. Options follow a second colon: ro or rw, docker's own mount options, volume-subpath=<path> and volume-chown=<option>:

```shell
dokku mariadb:mount lollipop /var/lib/dokku/data/storage/lollipop:/opt/extra:ro,z
```

A subpath mounts a directory within the source rather than the source itself. A docker volume mounted from a subpath needs Docker Engine 26.0 or newer, and takes no mount option but nocopy.

```shell
dokku mariadb:mount lollipop my-volume:/opt/extra:volume-subpath=uploads
```

A chown hands the mounted directory to a user before the container is made: herokuish, heroku, paketo, root or a uid. It is only taken for a host path inside the service's own directory.

```shell
dokku mariadb:mount lollipop /var/lib/dokku/services/mariadb/lollipop/extra:/opt/extra:volume-chown=heroku
```

The same can be said with flags instead:

```shell
dokku mariadb:mount lollipop /var/lib/dokku/data/storage/lollipop:/opt/extra --volume-readonly --volume-options z
```

Mounting the same source at the same directory again rewrites its options:

```shell
dokku mariadb:mount lollipop /var/lib/dokku/data/storage/lollipop:/opt/extra
```

Replace every mount the service has with the ones given:

```shell
dokku mariadb:mount --replace lollipop /srv/a:/opt/a:ro /srv/b:/opt/b
```

> NOTE: a mount reaches the container the next time one is built. mariadb:restart keeps the container it has, so use mariadb:stop and then mariadb:start on a service that is already running.

### remove one or all mounts from the service container

```shell
# usage
dokku mariadb:unmount [--all] <service> [<source:container-dir>...]
```

flags:

- `--all`: remove every mount the service has

Remove a mount, naming it the way it was mounted:

```shell
dokku mariadb:unmount lollipop /var/lib/dokku/data/storage/lollipop:/opt/extra
```

Remove every mount the service has:

```shell
dokku mariadb:unmount --all lollipop
```

> NOTE: the mount is removed from the container the next time one is built. mariadb:restart keeps the container it has, so use mariadb:stop and then mariadb:start on a service that is already running.

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

The connection tool only shows a prompt when it is given a terminal, which ssh allocates when run with -t. Without a terminal, statements are read from stdin instead.

```shell
dokku mariadb:connect lollipop < statements.txt
```

### enter or run a command in a running MariaDB service container

```shell
# usage
dokku mariadb:enter <service>
```

A shell can be opened against a running service. Filesystem changes will not be saved to disk.

> NOTE: disconnecting from ssh while running this command may leave zombie processes due to moby/moby#9098

```shell
dokku mariadb:enter lollipop
```

You may also run a command directly against the service. Filesystem changes will not be saved to disk.

```shell
dokku mariadb:enter lollipop touch /tmp/test
```

### expose a MariaDB service on custom host:port if provided (random port on the 0.0.0.0 interface if otherwise unspecified)

```shell
# usage
dokku mariadb:expose <service> <ports...>
```

Expose the service on the service's normal ports, allowing access to it from the public interface (`0.0.0.0`):

```shell
dokku mariadb:expose lollipop 3306
```

Expose the service on the service's normal ports, with the first on a specified ip address (127.0.0.1):

```shell
dokku mariadb:expose lollipop 127.0.0.1:3306
```

Expose the service on random ports on a single address, and only to clients in one network:

```shell
dokku mariadb:set lollipop expose-address 10.0.0.5
dokku mariadb:set lollipop expose-source-range 10.0.0.0/8
dokku mariadb:expose lollipop
```

### unexpose a previously exposed MariaDB service

```shell
# usage
dokku mariadb:unexpose <service>
```

Unexpose the service, removing access to it from the public interface (`0.0.0.0`):

```shell
dokku mariadb:unexpose lollipop
```

### reexpose a MariaDB service, applying its expose settings without restarting it

```shell
# usage
dokku mariadb:reexpose <service>
```

Apply a changed expose-address or expose-source-range to an exposed service, on the ports it is already exposed on:

```shell
dokku mariadb:set lollipop expose-source-range 10.0.0.0/8
dokku mariadb:reexpose lollipop
```

> NOTE: only the container publishing the service's ports is replaced, so the service keeps running, though connections made through the exposed ports are dropped. A service that is not exposed, or is not running, is refused.

### promote service <service> as DATABASE_URL in <app>

```shell
# usage
dokku mariadb:promote <service> [<app>]
```

If you have a mariadb service linked to an app and try to link another mariadb service another link environment variable will be generated automatically:

```
DOKKU_MARIADB_AQUA_URL=mysql://:ANOTHER_PASSWORD@dokku-mariadb-other-service:3306/other_service
```

You can promote the new service to be the primary one:

> NOTE: this will restart your app

```shell
dokku mariadb:promote other_service playground
```

This will replace `DATABASE_URL` with the url from other_service and generate another environment variable to hold the previous value if necessary. You could end up with the following for example:

```
DATABASE_URL=mysql://:ANOTHER_PASSWORD@dokku-mariadb-other-service:3306/other_service
DOKKU_MARIADB_AQUA_URL=mysql://:ANOTHER_PASSWORD@dokku-mariadb-other-service:3306/other_service
DOKKU_MARIADB_BLACK_URL=mysql://:SOME_PASSWORD@dokku-mariadb-lollipop:3306/lollipop
```

### start a previously stopped MariaDB service

```shell
# usage
dokku mariadb:start <service>
```

Start the service:

```shell
dokku mariadb:start lollipop
```

A service comes back on the version it was created with, or was last upgraded to, whatever version the plugin ships now. The image is fetched if the host no longer has it. A service that has never recorded a version and has no container left to read one from cannot be placed, and is reported rather than started on a guess. Use mariadb:upgrade to say which version it should run.

### stop a running MariaDB service

```shell
# usage
dokku mariadb:stop <service>
```

Stop the service and removes the running container:

```shell
dokku mariadb:stop lollipop
```

### pause a running MariaDB service

```shell
# usage
dokku mariadb:pause <service>
```

Pause the running container for the service:

```shell
dokku mariadb:pause lollipop
```

### graceful shutdown and restart of the MariaDB service container

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

- `-c|--config-options <string>`: extra arguments for the process the service container runs, not docker flags; use mount for mounts
- `-C|--custom-env <string>`: semi-colon delimited environment variables to start the service with
- `-i|--image <string>`: the image to upgrade the service to
- `-I|--image-version <string>`: the image version to upgrade the service to
- `-N|--initial-network <string>`: the initial network to attach the service to
- `--log-driver <string>`: the docker logging driver to run the service container with (default: the daemon's own)
- `--log-opt <strings>`: a comma-separated list of key=value docker log options for the service container
- `-m|--memory <int>`: container memory limit in megabytes, 0 for unlimited
- `-P|--post-create-network <strings>`: a comma-separated list of networks to attach the service container to after service creation
- `-S|--post-start-network <strings>`: a comma-separated list of networks to attach the service container to after service start
- `--restart <string>`: the docker restart policy to run the service container with (default: always)
- `-R|--restart-apps`: whether to stop and start the linked apps around the upgrade
- `-s|--shm-size <string>`: override shared memory size for the service docker container
- `--volume <stringArray>`: a host path or docker volume to mount into the service container, as <source>:<container-dir>[:<options>], repeatable
- `--wait-timeout <string>`: seconds to wait for the service to become ready (default: the datastore's own)

You can upgrade an existing service to a new image or image-version:

```shell
dokku mariadb:upgrade lollipop
```

This is the only command that changes the version a service runs. With no version named it moves to the newest the service's own major version ships, which leaves the data where it is.

```shell
dokku mariadb:upgrade lollipop --image-version 1.2.3
```

Moving across a major version has to be asked for by name, because it is not a tag change: the data is mounted somewhere different under the new one, and pointing the version back does not undo it. A service keeps the mounts it has unless --volume is passed, which replaces them, and each one is checked against the new container before the old one is taken away.

```shell
dokku mariadb:upgrade lollipop --volume /var/lib/dokku/data/storage/lollipop:/opt/extra:ro
```

A service keeps its memory limit unless --memory is passed, and --memory 0 removes it.

```shell
dokku mariadb:upgrade lollipop --memory 512
```

### Service Automation

Service scripting can be executed using the following commands:

### list all MariaDB service links for a given app

```shell
# usage
dokku mariadb:app-links [<app>]
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

- `-c|--config-options <string>`: extra arguments for the process the service container runs, not docker flags; use mount for mounts
- `-C|--custom-env <string>`: semi-colon delimited environment variables to start the service with
- `-N|--initial-network <string>`: the initial network to attach the service to
- `--log-driver <string>`: the docker logging driver to run the service container with (default: the daemon's own)
- `--log-opt <strings>`: a comma-separated list of key=value docker log options for the service container
- `-m|--memory <int>`: container memory limit in megabytes (default: unlimited)
- `-p|--password <string>`: override the user-level service password, for datastores that have one
- `-P|--post-create-network <strings>`: a comma-separated list of networks to attach the service container to after service creation
- `-S|--post-start-network <strings>`: a comma-separated list of networks to attach the service container to after service start
- `--restart <string>`: the docker restart policy to run the service container with (default: always)
- `-r|--root-password <string>`: override the root-level service password, for datastores that have one
- `-s|--shm-size <string>`: override shared memory size for the service docker container
- `--volume <stringArray>`: a host path or docker volume to mount into the service container, as <source>:<container-dir>[:<options>], repeatable
- `--wait-timeout <string>`: seconds to wait for the service to become ready (default: the datastore's own)

You can clone an existing service to a new one:

```shell
dokku mariadb:clone lollipop lollipop-2
```

The new service starts from the settings of the one it copies: its config options, custom env, memory, shm size, networks, log driver, log options, restart policy, mounts and backup keyserver. A flag passed to clone overrides that one setting, and a flag passed empty clears it:

```shell
dokku mariadb:clone lollipop lollipop-2 --restart no --custom-env ""
```

The password, exposed ports, links and backup credentials, schedule and encryption are not copied. The clone's passwords are generated unless they are given.

```shell
dokku mariadb:clone lollipop lollipop-2 --password <password>
```

### check if the MariaDB service exists

```shell
# usage
dokku mariadb:exists <service>
```

Here we check if the lollipop mariadb service exists.

```shell
dokku mariadb:exists lollipop
```

### check if the MariaDB service is linked to an app

```shell
# usage
dokku mariadb:linked <service> [<app>]
```

Here we check if the lollipop mariadb service is linked to the `playground` app.

```shell
dokku mariadb:linked lollipop playground
```

### list all apps linked to the MariaDB service

```shell
# usage
dokku mariadb:links <service>
```

List all apps linked to the `lollipop` mariadb service.

```shell
dokku mariadb:links lollipop
```

Renaming an app moves its link onto the new name, and cloning an app links the clone as well as the original.

### Data Management

The underlying service data can be imported and exported with the following commands:

### import a dump into the MariaDB service database

```shell
# usage
dokku mariadb:import <service> [-f|--file <path>] [-- <import-args...>]
```

flags:

- `-f|--file <string>`: a file on the dokku host to import instead of reading stdin

Import a datastore dump:

```shell
dokku mariadb:import lollipop < data.dump
```

A dump that is already on the dokku host can be imported with --file. The path is on the dokku host, not on the machine running ssh.

```shell
dokku mariadb:import lollipop --file /var/lib/dokku/data/storage/data.dump
```

Arguments after -- are passed to the tool that loads the dump, in place of the import-args property:

```shell
dokku mariadb:import lollipop -- <import-args...> < data.dump
```

The import-args property holds the arguments every import into the service is made with, a clone's included:

```shell
dokku mariadb:set lollipop import-args -- "<import-args...>"
```

### export a dump of the MariaDB service database

```shell
# usage
dokku mariadb:export <service> [-f|--file <path>] [--force] [-- <export-args...>]
```

flags:

- `-f|--file <string>`: a file on the dokku host to export to instead of writing stdout
- `--force`: replace the file named with --file if it already exists

By default, datastore output is exported to stdout:

```shell
dokku mariadb:export lollipop
```

You can redirect this output to a file:

```shell
dokku mariadb:export lollipop > data.dump
```

A dump can be written to a file on the dokku host with --file. The path is on the dokku host, not on the machine running ssh.

```shell
dokku mariadb:export lollipop --file /var/lib/dokku/data/storage/data.dump
```

A file that already exists is not overwritten unless --force is given:

```shell
dokku mariadb:export lollipop --file /var/lib/dokku/data/storage/data.dump --force
```

Arguments after -- are passed to the tool that makes the dump, in place of the export-args property:

```shell
dokku mariadb:export lollipop -- <export-args...>
```

The export-args property holds the arguments every export, backup and clone of the service is made with:

```shell
dokku mariadb:set lollipop export-args -- "<export-args...>"
```

### Backups

Datastore backups are supported via AWS S3 and S3 compatible services like [minio](https://github.com/minio/minio).

You may skip the `backup-auth` step if your dokku install is running within EC2 and has access to the bucket via an IAM profile. In that case, use the `--use-iam` option with the `backup` command.

If both passphrase and public key forms of encryption are set, the public key encryption will take precedence.

The underlying core backup script is present [here](https://github.com/dokku/docker-s3backup/blob/main/backup.sh).

Scheduled backups are added to the dokku crontab, and are listed by `dokku cron:list --global`.

Backups can be performed using the backup commands:

### set up authentication for backups on the MariaDB service

```shell
# usage
dokku mariadb:backup-auth <service> <aws-access-key-id> <aws-secret-access-key> <aws-default-region> <aws-signature-version> <endpoint-url>
```

Setup s3 backup authentication:

> NOTE: each call replaces the stored credentials as a whole, so a region, signature version or endpoint url that is not passed is removed

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

### remove backup authentication for the MariaDB service

```shell
# usage
dokku mariadb:backup-deauth <service>
```

Remove s3 authentication:

```shell
dokku mariadb:backup-deauth lollipop
```

### create a backup of the MariaDB service to an existing s3 bucket

```shell
# usage
dokku mariadb:backup <service> <bucket-name> [-u|--use-iam]
```

flags:

- `-u|--use-iam`: use the IAM profile associated with the current server

Backup the `lollipop` service to the `my-s3-bucket` bucket on `AWS`:

```shell
dokku mariadb:backup lollipop my-s3-bucket --use-iam
```

Restore a backup file (assuming it was extracted via `tar -xf backup.tgz`):

```shell
dokku mariadb:import lollipop < backup-folder/export
```

### set encryption for all future backups of MariaDB service

```shell
# usage
dokku mariadb:backup-set-encryption <service> <passphrase>
```

Set the GPG-compatible passphrase for encrypting backups for backups:

```shell
dokku mariadb:backup-set-encryption lollipop
```

Public key encryption will take precendence over the passphrase encryption if both types are set.

### set GPG Public Key encryption for all future backups of MariaDB service

```shell
# usage
dokku mariadb:backup-set-public-key-encryption <service> <public-key-id>
```

Set the `GPG` Public Key for encrypting backups:

```shell
dokku mariadb:backup-set-public-key-encryption lollipop
```

The <public-key-id> is fetched from `keyserver.ubuntu.com`, unless the service names another one with the backup-keyserver property:

```shell
dokku mariadb:set lollipop backup-keyserver hkp://keys.example.com
```

### unset encryption for future backups of the MariaDB service

```shell
# usage
dokku mariadb:backup-unset-encryption <service>
```

Unset the `GPG` encryption passphrase for backups:

```shell
dokku mariadb:backup-unset-encryption lollipop
```

### unset GPG Public Key encryption for future backups of the MariaDB service

```shell
# usage
dokku mariadb:backup-unset-public-key-encryption <service>
```

Unset the `GPG` Public Key encryption for backups:

```shell
dokku mariadb:backup-unset-public-key-encryption lollipop
```

### schedule a backup of the MariaDB service

```shell
# usage
dokku mariadb:backup-schedule <service> <schedule> <bucket-name> [-u|--use-iam]
```

flags:

- `-u|--use-iam`: use the IAM profile associated with the current server

Schedule a backup:

> 'schedule' is a crontab expression, eg. "0 3 * * *" for each day at 3am, or a descriptor such as "@daily". A schedule cron cannot run is refused.
> the backup is added to the dokku crontab through the cron-entries plugin trigger, so it is listed by "dokku cron:list --global" and its output is appended to /var/log/dokku/mariadb.log
> NOTE: dokku only writes a crontab when the global scheduler or at least one app uses the docker-local scheduler, so a scheduled backup does not run on a host that only uses k3s or null

```shell
dokku mariadb:backup-schedule lollipop "0 3 * * *" my-s3-bucket
```

Schedule a backup and authenticate via iam:

```shell
dokku mariadb:backup-schedule lollipop "0 3 * * *" my-s3-bucket --use-iam
```

### cat the crontab line of the scheduled backup for the service

```shell
# usage
dokku mariadb:backup-schedule-cat <service>
```

Cat the crontab line of the scheduled backup for the service:

```shell
dokku mariadb:backup-schedule-cat lollipop
```

### unschedule the backup of the MariaDB service

```shell
# usage
dokku mariadb:backup-unschedule <service>
```

Remove the scheduled backup from the dokku crontab:

```shell
dokku mariadb:backup-unschedule lollipop
```

### Limiting where and to whom a service is exposed

An exposed service's ports are published on every interface unless they are given an address of their own. To publish them on one address instead, set the service's `expose-address` property with `dokku mariadb:set`, and to accept connections only from clients in one IP address or CIDR, set its `expose-source-range` property. Either reaches a running service with `dokku mariadb:reexpose`, which leaves the service running.

Only one source range can be given. The range is checked against the address a connection reaches the service from, which for a connection to the exposed port on the loopback interface, or an IPv6 connection to a service network without IPv6, is the docker network's gateway rather than the client, so with a range that leaves the gateway out, connecting to `127.0.0.1` from the dokku host itself is refused.

### Passing extra arguments to export and import

Arguments given to `export` or `import` after `--` are appended to the ones the datastore's own tool is run with, for that run alone. To use them every time, set the service's `export-args` or `import-args` property with `dokku mariadb:set`, giving the value after `--` so that its leading dash is not read as a flag. The property is split the way a shell would split it, so an argument with a space in it is quoted, and a variable in it is refused rather than expanded.

Arguments given after `--` replace the property rather than adding to it. Backups and clones are made with the property, and a clone is given the source's.

### Waiting for a service to become ready

A service is waited on until it answers on its port after it is created, cloned, started, restarted, upgraded or exposed. If it takes longer than that to start - on a slow host, or with an image that does more on its first boot - the command fails with `ERROR: unable to connect`.

To wait longer for every mariadb service on the host, set the `MARIADB_WAIT_TIMEOUT` environment variable to a number of seconds. To wait longer for a single service, set its `wait-timeout` property with `dokku mariadb:set` or pass `--wait-timeout` to `create`, `clone` or `upgrade`. The service's own setting is used first, then the environment variable, then the datastore's default.

### Reserved service names

A service's database is named after the service, with hyphens replaced by underscores. So that an app is never handed a database MariaDB keeps for itself, `dokku mariadb:create` and `dokku mariadb:clone` refuse a name that is, or whose database would be, one of `information_schema`, `mysql`, `performance_schema`, `sys`, in any case. A service that already has such a name is not affected.

### Disabling `docker image pull` calls

If you wish to disable the `docker image pull` calls that the plugin triggers, you may set the `MARIADB_DISABLE_PULL` environment variable to `true`. Once disabled, you will need to pull the service image you wish to deploy as shown in the `stderr` output.

Please ensure the proper images are in place when `docker image pull` is disabled.
