Load Balancing Apache Camel routes with HashiCorp Consul
==========================================

This example show how to use Camel with HashiCorp Consul.  It follows the ServiceCall Enterprise Integration Pattern (EIP), and makes use of Spring Boot.

This example includes two maven modules:

 - services that exposes a number of services
 - consumer that consumes services

## Prerequisites

1. Docker
2. Java 11+
3. Maven 3.8+

## Setup

Via the CLI,
1. Pull Consul's official Docker image: `docker pull consul`
1. Start the Consul agent as a server: `docker run -d -e CONSUL_BIND_INTERFACE=eth0 consul agent -dev -join=172.17.0.2 -ui -p 8500:8500`
1. Start the first Consul client: `docker run -d -e CONSUL_BIND_INTERFACE=eth0 consul agent -dev -join=172.17.0.2 -ui`
1. Start the second Consul client: `docker run -d -e CONSUL_BIND_INTERFACE=eth0 consul agent -dev -join=172.17.0.2 -ui`

The consumer is configured in the src/main/resources/application.properties in which we blacklist some services for being discovered and we add some additional services not managed by consul

```
    # Configure service filter
    camel.cloud.service-filter.blacklist[service-1] = localhost:9012

    # Configure additional services
    camel.cloud.service-discovery.services[service-2] = localhost:9021,localhost:9022,localhost:9023
```

## Build

You can build this example using

```
    mvn compile
```

## Run the example

Using multiple shells:

 - start consul:

```
  $ cd services
  $ src/main/bash/consul-run-linux.sh      (for linux)
  $ src/main/bash/consul-run-osx.sh        (for mac osx)
```

 - start the service-1 service group:

```
  $ cd services
  $ mvn spring-boot:run -Dspring-boot.run.profiles=service-1
```

  - start the service-2 service group:

```
  $ cd services
  $ mvn spring-boot:run -Dspring-boot.run.profiles=service-2
```

  - start the consumer

```
  $ cd consumer
  $ mvn spring-boot:run
```

## Test the example:

In a new shell:

```
  $ curl localhost:8080/camel/serviceCall/service1
  Hi!, I'm service-1 on camel-1/route1
  $ curl localhost:8080/camel/serviceCall/service2
  Hi!, I'm service-1 on camel-1/route2
```

If you keep calling the http endpoint you'll notice they are consumed using a round robin policy and that one of the services registered in consul is not taken into account according to the blacklist.

## Web console

You can open the Consul web console

```
     http://localhost:8500/ui
```

Where you can find information about the services and its state.
     
## Help and contributions

If you hit any problem using Camel or have some feedback, then please
https://camel.apache.org/support.html[let us know].

We also love contributors, so
https://camel.apache.org/contributing.html[get involved] :-)

The Camel riders!
