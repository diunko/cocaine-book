
# Framework

## What it does and helps to do

So one can interact with cocaine environment and all of it's
components by exchange of messages. To help developers develop their
apps quickly and be compliant with requirements of cocaine
environment, there are frameworks for languages. Basicly, framework
gives the developer means to interact as a client with any service, or
helps to run her code in the cloud. It helps to interact with various
conditions such as session handling and lifetime events like termination.

## Client

Client component of a framework allows developer to interact with any
cocaine service. It resolves certain service, provides means to
conveniently call it methods, and means to handle response stream,
means to interpret streams according to developer's needs.

## Worker

Worker component of a framework gives a developer means to turn her
code into scaling app, and to run it compliantly under the management of
cocaine-runtime.

## Worker requirements

Here are requirements for any code that wants to implement behaving
worker under cocaine-runtime management.

1. Overview
1. Start-up
1. Handling sessions
1. Handling heartbeats
1. Handling and signaling termination


