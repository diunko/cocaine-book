
# Intro

## Actor Model

Actor model assumes that everything is an actor.

According to AA, actors have following properties.


create
send
designate how to handle the next message




1. Actor has an address.
1. Actor can spawn another actor
1. Actor can send and receive messages. Actor send messages only
actores, whose address it knows.

## Cocaine Actors

Cocaine doesn't bring all of Actors properties into life. In Cocaine
terms, Service is an Actor.

Service has an address. It is called an endpoint and it can be a tcp
address or a unix socket pathname.

Service can spawn another service.

Service can receive messages. It happens as follows.

Client connects to Service's endpoint. Client and Service exchange
framed messages using the socket.

Messages are formed using msgpack. A message is a tuple (array in
msgpack terms) of: message_type_id (sometimes called method_id),
session_id, and a tuple (array) of arguments.

`method_id`s are specified in corresponding `protocol`s.




