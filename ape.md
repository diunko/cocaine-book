
# APE

## overview

Currently `ape` is a simple bash script. Using ssh authorization, git,
and cocaine-tool, it solves task of per-user app management in cocaine
cloud. It provides user means to build application's docker image,
start/stop applications, and otherwise manage applications.

## operation

Firstly, `ape` operates as a git remote, and builds application image
when new changes are pushed to the remote.
Secondly, `ape` provides set of commands.

### commands

`start <app-name> <treeish>` starts app version and adds it to app
routing group

`stop <app-name> <treeish>` stops app version and removes it from app
routing group

`status` reports status of all user's apps: shows routing, running
apps and app versions, shows uploaded apps

`remove <app-name> <treeish>` stops app version, removes it from
routing group, removes app and app image from cloud

`adduser <username>` adds a line with user's ssh key to
`authorized_keys` file

### authorization

`ape` uses ssh authorization via authorized_keys file and it's `command`
capability, which is described in `authorized_keys` manual.
Every line of `authorized_keys` file contains key, command to
run (ape script) and environment variables like user login and key
fingerprint. 

When user logins with ssh as ape user and proper ssh key, ape script
is invoked with user login as environment variable and all
command-line parameters of ssh command.

## basic container

Scripts to build basic container are in (buildstep)[ repository: 

## buildpack

## setup

## usage


