# dokku mariadb (beta)

Official mariadb plugin for dokku. Currently installs mariadb 10.0.21.

## requirements

- dokku 0.3.25+
- docker 1.6.x

## installation

```
cd /var/lib/dokku/plugins
git clone https://github.com/dokku/dokku-mariadb.git mariadb
dokku plugins-install-dependencies
dokku plugins-install
```

## commands

```
mariadb:alias <name> <alias>     Set an alias for the docker link
mariadb:clone <name> <new-name>  NOT IMPLEMENTED
mariadb:connect <name>           Connect via mariadb to a mariadb service
mariadb:create <name>            Create a mariadb service
mariadb:destroy <name>           Delete the service and stop its container if there are no links left
mariadb:export <name>            Export a dump of the mariadb service database
mariadb:expose <name> [port]     Expose a mariadb service on custom port if provided (random port otherwise)
mariadb:import <name> <file>     NOT IMPLEMENTED
mariadb:info <name>              Print the connection information
mariadb:link <name> <app>        Link the mariadb service to the app
mariadb:list                     List all mariadb services
mariadb:logs <name> [-t]         Print the most recent log(s) for this service
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

# get connection information as follows
dokku mariadb:info lolipop

# lets assume the ip of our mariadb service is 172.17.0.1

# a mariadb service can be linked to a
# container this will use native docker
# links via the docker-options plugin
# here we link it to our 'playground' app
# NOTE: this will restart your app
dokku mariadb:link lolipop playground

# the above will expose the following environment variables
#
#   DATABASE_URL=mariadb://mariadb:SOME_PASSWORD@172.17.0.1:3306/lolipop
#   DATABASE_NAME=/lolipop/DATABASE
#   DATABASE_PORT=tcp://172.17.0.1:3306
#   DATABASE_PORT_3306_TCP=tcp://172.17.0.1:3306
#   DATABASE_PORT_3306_TCP_PROTO=tcp
#   DATABASE_PORT_3306_TCP_PORT=3306
#   DATABASE_PORT_3306_TCP_ADDR=172.17.0.1

# you can customize the environment
# variables through a custom docker link alias
dokku mariadb:alias lolipop MARIADB_DATABASE

# you can also unlink a mariadb service
# NOTE: this will restart your app
dokku mariadb:unlink lolipop playground

# you can tail logs for a particular service
dokku mariadb:logs lolipop
dokku mariadb:logs lolipop -t # to tail

# finally, you can destroy the container
dokku mariadb:destroy lolipop
```

## todo

- implement mariadb:clone
- implement mariadb:import
