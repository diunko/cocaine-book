
# Installation
Here are described various ways to obtain working cocaine
environment. If you want to taste it on windows or macos, you should
go for vagrant image. For development usage see setting development
environment in nodejs-dev-env.md.

## vagrant image

Install vagrant and needed plugins as described in README from
cocaine-vagrant repository. Then,

```
git clone git@github.yandex-team.ru:diunko/cocaine-vagrant.git
vagrant up
```

It installs yandex repositories, install cocaine debs, uatraits,
geobase and other services plugins, configures them.
Adds `ape` user, and in it's home directory installs and configures
ape script.

Afterwards you have to add your public key to the authorized_keys of
this user:

```
vagrant ssh
sshkeyctl user add ape <yourlogin> << EOF
<paste your public key here>
EOF
> user added ok
```

Now, you can add remote `ssh://ape@localhost:2222/appname` to your local
repository and deploy your app to your local cocaine with
`git push <remotename> master`.

## your one-node instance cocaine
Basically, to install cocaine on your server, you have to follow steps
from default.rb in cocaine recipe. You can do it by hand or using chef-solo.


## your own cloud
Set up every node according one-node procedure.

Set up shared elliptics storage.
TODO

Configure multicast group for announces on every node.
TODO

Configure IPVS gateway on the front-node.
TODO

Configure cocaine-proxy and nginx.
TODO

