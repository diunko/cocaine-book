
# Overview

## Big Picture Of What Is Where

Typical installation would look like this.

`<nice_image.png>`

Node server runs cocaine-runtime with `node` and `locator`
services. `node` service downloads and start

LoadBalancer runs nginx, cocaine-http-proxy, and cocaine-runtime with IPVS
gateway plugin and (meta) `locator`.

Storage's are nodes with elliptics. 

DockerRepository is a Docker repository. App images are pushed there
after they are built, and pulled from there when they are deployed.

DockerBuild is a node where apps are pushed to, and it builds
containers, and pushes them to docker repository.

## What Happens And What Everybody Does

### Request to the started, running app
nginx, cocaine-http, cocaine-runtime, IPVS, cocaine-runtime,
service, app, worker.

The http request comes in to nginx. Nginx can tell somehow which app
is it's destination. It sets header `x-cocaine-service` with the name
of an app, and header `x-cocaine-handle` with some predetermined
handle name (or event name) which your worker code is supposed to
actually handle (see Apps in intro). Most of the time it will be just
"http".

With these headers, nginx sends http request to
cocaine-http-proxy. Proxy resolves service from the x-cocaihe-service
header, connects to it (it may cache connections), and passes request
in `enqueue` message. It encodes request as
`["GET", <url>, "1.1", [ [header, value], [header, value], ...]]`
It expects the first chunk of the response stream to be 
`[200, [[header,value], [header,value],...]]`
and all other stream chunks to carry response body chunks.

When request arrives into `cocaine-runtime` into invocation service,
it's transformed into the following sequence of messages:
[`invoke`, sid, [eevnt]], `[chunk, sid, [http-request-chunk]]`,
`[choke, sid, []]`

Here, cocaine-runtime does the Load Balancing (see the next section
for details) and chooses the worker process among the running ones,
starts a new one, or drops the message returning error back to
cocaine-http-proxy. Let's assume all went well.

Next, worker receives invoke message with event, receives chunks with
request, it is supposed to reply with a correctly formatted first
header chunk and the other chunks that form a http response body. All
messages are get proxied back to cocaine-http-proxy which gets them
back to nginx, and then they get further back to client.


### Node joins cluster
Announce, synchronize.

### App is uploaded to the cluster.
Let's consider the case where the app is packaged as a tarball. 
cocaine-tool puts the tarball into storage["apps",<apname>]. 
cocaine-tool puts app's manifest into
storage["manifests",<appname>]
And that's pretty much all.

### App is started.
1. cocaine-tool reads storage["runlists"], adds appname: profile record
to it and stores it back.
1. cocaine-tool, for every node of a cluster, sendsa "start_app"
message with the name of the app being started to it's service `node`.
1. cluster Node, when it receives "start_app" message, gets app's
manifest, tarball, unpacks tarball to the local spoll directory.
1. cluster Node starts "engine" which listens to incoming connection
on local socket designated for app worker processes.
1. cluster Node starts designated invocation service, which listens
for incoming requests, with the name of started app, and registers it
with local locator.
1. local locator sends a chunk with app service metadata to all of its
established synchronize streams.

### First, cold Request for a recently started app.
1. as in [A], request arrives to the cocaine-runtime's balancing
procedure. When it finds, that there's not enough running worker
processes (actually, there's zero of them if the request is the first
one), it does usual balancing job of spawning workers (see Spawning)
and sends request to worker as usual (see [A]).

### Spawning Slave Worker Process

App pause.



## How The Load Is Balanced

### On The LoadBalancer

### In The App Instance

