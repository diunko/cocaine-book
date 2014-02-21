
# [Введение](intro.md)

## Actor Model
## Cocaine Messages and Actors
## Cocaine RPC Protocol
## Locator
## Apps and Services


# [Setup Overview](setup-overview.md)

## Big Picture
## Node Box
## LoadBalancer Box
## DockerRepository Box
## Dockerbuild Box
## Elliptics cluster
## ElasticSearch cluster


# [Processes Overview](processes-overview.md)

## Regular App Deployment
### process isolate
### docker isolate

## Deployment Through ape.sh

### Build App Container
### Deploy App Container
### App Request Handling


# [Services Overview](services-overview.md)

## Storage
## Logging
## Geobase
## Uatraits
## Langdetect


# [Technical details](technical-details.md)

## Frameworks

### Worker lifecycle
### Client requirements


## cocaine-tool

## cocaine-http-proxy

## IPVS plugin

## [APE build flow details](ape-details.md)

## NodeJS

### [Framework](nodejs-framework.md)

#### Overview

#### Worker Details
#### Client Details
#### Service


### [Development Environment](nodejs-development-environment.md)

#### cocaine-vagrant
#### building on linux
#### building on os x
#### setting up nginx and rewrites

### [Examples, Guides and Best Practices](nodejs-guides.md)

#### simple app
#### use simple service: storage
#### use infrastructure services
#### use services from within your app
#### use app as a service
#### watching your logs

#### Best Practice: Logging
#### Best Practice: handling client errors
#### Best Practice: handling worker lifecycle events


