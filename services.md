
# Services

## What is a Service

So, a Service is a piece of c++ code that is run in it's own thread,
that implements a bunch of methods available through RPC
protocol. Current implementation of Service class supports the
following message exchange.

Service expects incoming messages in the form of

`[<method_id>, <session_id>, [<arguments>]]`

Here, `method_id` is a numeric id of a particular service method in
service methods table (as defined in protocol<protocol_tag>::type
typedef in messages.hpp),
`session_id`, as usual, is a client-generated numeric id of a session
unique within a channel, and <arguments> is a tuple of a predetermined
structure specified in corresponding protocol::tuple_type typedef.

The response to the call is conveyed in form of a stream, i.e. zero or
more chunk messages, followed by choke or error
message. Interpretation of data found in chunks is upto calling
client. Usually, it is some kind of a msgpack-packed structure.

## How is it run

Service is configured according to corresponding section in
cocaine-runtime configuration file. For every key of "services"
section the corresponding service is attempts to start. In case of
failure of any of configured services, the cocaine-runtime fails to
start. Every service is registered within Locator service, and
published on it's designated endpoint.

## Core Services

### locator
Locator is a pseudo-service, in a sense that it can't be resolved with
any of cocaine mechanics, and client has to know somehow it's
parameters beforehand. Any other service, run by cocaine-runtime, is
registered within the locator, and available for resolution through
it.

#### methods
1. resolve
arguments: `[<string:service_name>]`
client supplies `service_name`, which is a service name which it wants
to resolve and afterwards to connect to.

result:
`[<endpoint>, <protocol_version>, {0: method_name0, 1: method_name1}]`

`<endpoint>` is either `string:/path/to/unix.sock` or
`[string:host, int: port]` correspondingly for unis socket or tcp
socket.

`<protocol_version>` is currently 1 for any protocol. 

The third element in the result in a mapping from numeric method ids
to corresponding method names.


1. synchronize
arguments: `[]` -- no arguments expected

result: a stream. Each chunk in the response stream is sent as soon
as new updates in running services happen, and each chunk carries the
payload of the following form:

`{service_name1: <resolve_result1>, service_name2: <resolve_result2>}`

where `resolve_resultX` is the result of `service_nameX` resolution,
as it comes out of `resolve` method.


### node
Service node is used to manage running apps on nodes of cluster.
At start, it reads it's configured runlist from storage, and starts
all apps listed there (as described in Start App section in Overview)
with corresponding profiles.
`start_app` message used to start app without reading the runlist,
`pause_app` message used to stop running app without write to the
runlist. `list` message returns all started apps known to current
cluster node.


#### methods
1. start_app
   arguments: `[{app_name1: profile_name1, app_name2: profile_name2}]`
   starts applications with corresponding profiles, i.e. app named
   app_name1 starts with profile named profile_name1
   result: empty stream

1. pause_app
   arguments: `[app_name1, app_name2, ..]`
   for every app_name stops corresponding app's instances on current
   node.
   result: empty stream

1. list
   arguments: `[]`
   lists all known apps running in the cloud.
   result: empty stream
   
#### configuration

### logging
### storage


## Other Services

### elliptics storage
Has a regular storage methods, extended with methods to write to
memory-only storage. Detailed description see in elliptics-storage.md.

### uatraits
It's purpose is to detect browser's details and capabilities given all
the relevant request's headers. It outputs result in form of. Detailed
description see on wiki at [TODO].

### geobase
Is a service wrapping geobase bindings with all of the corresponding
methods. See full reference at [TODO].

### urlfetch
Is a service to make various http requests, incluing POST and others.
See full reference at [TODO].


