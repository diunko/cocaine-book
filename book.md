# DONE [Intro](intro.md) <code>[5/5]</code>

-   [X] Actor Model
-   [X] Cocaine Messages and Actors
-   [X] Cocaine RPC Protocol
-   [X] Locator
-   [X] Apps and Services

# TODO [Setup Overview](setup-overview.md) <code>[6/7]</code>

-   [-] Big Picture <code>[1/2]</code>
    -   [ ] diagram
    -   [X] description
-   [X] Node Box
-   [X] LoadBalancer Box
-   [X] DockerRepository Box
-   [X] Dockerbuild Box
-   [X] Elliptics cluster
-   [X] ElasticSearch cluster

# TODO [[<processes-overview.md][Processes> Overview] <code>[2/3]</code>

## DONE Regular deployment process <code>[2/2]</code>

### DONE process isolate <code>[5/5]</code>

-   [X] upload
-   [X] start
-   [X] handle requests
-   [X] stop
-   [X] remove

### DONE docker isolate <code>[5/5]</code>

-   [X] upload
-   [X] start
-   [X] handle requests
-   [X] stop
-   [X] remove

## DONE Deployment Through ape.sh <code>[7/7]</code>

-   [X] prerequisites
-   [X] version management notes
-   [X] upload
-   [X] start
-   [X] request handling
-   [X] stop
-   [X] remove

## TODO Request Handling And Balancing From Start To End <code>[0/3]</code>

-   [ ] at LoadBalancer
-   [ ] at Node
-   [ ] at Worker

# TODO [Services Overview](services-overview.md) <code>[0/7]</code>

## TODO Recap Of Basic Notions

## TODO Logging

## TODO Storage

## TODO Urlfetch

## TODO Geobase

## TODO Uatraits

## TODO Langdetect

# TODO Customization and Plugin Types <code>[0/5]</code>

-   [ ] isolate
-   [ ] gateway
-   [ ] service
-   [ ] storage
-   [ ] logger

# TODO [Technical Details](technical-details.md) <code>[0/5]</code>

## TODO Frameworks

### TODO Worker lifecycle

### TODO Client requirements

## Request balancing

-   [ ] IPVS
-   [ ] engine::balance

## TODO Special Capabilities

-   [ ] Routing groups

## TODO cocaine-tool

## TODO cocaine-http-proxy

## TODO IPVS plugin

# DONE [APE.sh Details](ape-details.md) <code>[3/3]</code>

## DONE buildstep (basic container)

## DONE heroku-buildpack-nodejs

## DONE ape <code>[7/7]</code>

-   [X] git-\*
-   [X] git-hook
-   [X] build
-   [X] status
-   [X] start
-   [X] stop
-   [X] delete

# TODO [Docker lifecycle quick overview](docker-overview.md)

-   [ ] pull
-   [ ] run
-   [ ] commit
-   [ ] push
-   [ ] Dockerfile

# TODO NodeJS <code>[0/3]</code>

## TODO [Framework](nodejs-framework.md) <code>[0/2]</code>

### TODO Overview

### TODO Reference

-   [ ] FSM
-   [ ] Worker Details
-   [ ] Client Details
-   [ ] Service
-   [ ] Sessions
-   [ ] Service wrappers

## TODO [development Environment](nodejs-development-environment) <code>[0/4]</code>

### TODO cocaine-vagrant

### TODO building on linux

### TODO building on os x

### TODO setting up nginx and rewrites

## TODO [Examples, Guides and Best Practices](nodejs-guildes.md) <code>[0/9]</code>

### TODO simple app

### TODO use simple service: storage

### TODO use infrastructure services

### TODO use services from within your app

### TODO use app as a service

### TODO watching your logs

### TODO Best Practice: Logging

### TODO Best Practice: handling client errors

### TODO Best Practice: handling worker lifecycle events

    aoidjfaoidfjo