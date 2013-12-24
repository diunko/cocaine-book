
# cocaine-tool

## What it does

`cocaine-tool` is a cli tool which gives user a direct access to
various entities in Cocaine cloud. It allows to manage and inspect
many of cloud's entities:

* uploaded apps
* running apps
* runlists
* routing groups

Also, it allows to manually resolve services and call service's methods.


## Cases and Examples

Upload app, start it, get it's info.
```
cocaine-tool app upload --name <app-name> --manifest <path/to/manifest> --package <path/to.tgz>
```
Here, app-name is an identifier by which app would be known to the
cloud. Manifest 



Modify runlist.

See routing info.

