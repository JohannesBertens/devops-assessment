# DevOps Assessment Documentation

## Background

Provided are two different web API's for a fictional airport application, only GET requests are supported:

* country-service
  * version 1.0.1: a service which returns basic information about countries
    * Endpoint: `/countries` to get a full list of countries
    * Endpoint: `/countries/<query>` to search for country by name / ISO code.
* airport-service
  * version 1.0.1: a service which returns information about airports with country codes
    * Endpoint: `/airports` to get a full list of airports and their runways
    * Endpoint: `/airports/<query>` to get a list of airports based on country code (e.g.: "NL")
  * version 1.1.0: a service which returns information about airports with country codes
    * Endpoint: `/airports?full=[0|1]` returns a summary or all details of all airports, depending on the value of full.
    * Endpoint: `/airports/<id>` returns the full information of an airport based on its identifier. E.g.: `/airports/EHAM` returns all information for Schiphol.
    * Endpoint: `/search/<qry>` returns a list of airports based on a country code search.

In addition to their application-specific endpoints all versions of both services expose the following two endpoints:

* /health/live returns 
  * 200 when the HTTP server is up.
* /health/ready returns
  * 200 when the service is done initializing and ready to serve requests, 
  * 503 when the service is still initializing.

The above-mentioned services should run in isolated environments and both endpoints are combined and exposed using a reverse-proxy and/or load-balancer.

## Requirements

Choose any technology stack, meet the following requirements:

* The entire stack should be able to run locally on a developer's machine
* The country- and airport-service run isolated from each other
  * No inter-communication between the two services is possible
  * No direct communication from the "outside world" is possible with the two services
* A reverse-proxy and/or load-balancer exposes the services on port 8000
* Initially, airports version 1.0.1 is deployed
  * An update to the airports service from version 1.0.1 to version 1.1.0 can be triggered at the code review without causing a service interruption.

Bonus points for increased use of automation, e.g.:
Automation of the deployment using a CI/CD tool.
Single click/command startup of the initial stack.

## Architecture and Design

### Option A: (virtual) machines
The most simple deployment possible: running the services on a single VM or machine connected directly to the internet, is both unwanted and violates the isolation restriction.

The next most simple deployment would be to run the services on two different (virtual) machines, with a third (virtual) machine that runs the reverse-proxy or load-balancer. The two machines running the services would not be connected to each other or the "outside world". The third machine would be connected to each machine running the service seperately and to the "outside world".

This is illustrated in the following figure:

<< insert figure with 3 (virtual) machines + VNET configurations >>


This option can be run locally using Virtual Machines and be setup using automation, for example with VirtualBox [https://www.virtualbox.org/], Ansible [https://www.ansible.com/] and Vagrant [https://www.vagrantup.com/].

When running the services multiple times, in combination with using a load-balancer, it is possible to bring up a new version of a service without causing a service interruption. An example of a load-balancer (and reverse-proxy) which supports this is HAProxy [http://www.haproxy.org/].

Having the upgrade performed automatically when a check-in is done on the repository, is possible with Octopus Deploy [https://octopus.com/], which works with any programming language and most cloud providers. It also works locally when an agent is installed on the development environment.

### Option B: Containerization with Docker
While the use of VirtualBox allows for creation of multiple virtual machines locally, it is time-consuming to keep these machines up-to-date and memory-consuming to run multiple instances of them.

This is where Docker shines, simplifying the creation and running of instances or "containers" of code. Creating a new (isolated) environment where your code runs is done by using a `Dockerfile`. For both API services, the Dockerfile is pretty similar, this is the one for Airports:
```
FROM openjdk:12-alpine

RUN mkdir -p /usr/src/app
COPY ./bin/airports-assembly-1.0.1.jar /usr/src/app
WORKDIR /usr/src/app

CMD ["java", "-jar", "airports-assembly-1.0.1.jar"]
```

And this is the one for Countries:
```
FROM openjdk:12-alpine

RUN mkdir -p /usr/src/app
COPY ./bin/countries-assembly-1.0.1.jar /usr/src/app
WORKDIR /usr/src/app

CMD ["java", "-jar", "countries-assembly-1.0.1.jar"]
```

There are (by default) no ports exposed to any other container, this is done separately by adding containers to networks. The only container we run with an exposed port, is the container for HAProxy: our reverse-proxy. It has an even simpler Dockerfile:
```
FROM haproxy:1.8-alpine
COPY haproxy.cfg /usr/local/etc/haproxy/haproxy.cfg
```

The config file is where the rules are for reaching both the Airports and the Countries APIs.

Using Docker networks, the HAProxy can reach both the Airports and Country instances by using the built-in DNS.

Creating these three instances is done by using `docker-compose` and a `docker-compose.yml` file, which describes for each container how to start it and what network it is part of.

This configuration only runs each container once, adding more running instances can be done by using `Docker Services` and `Docker Stacks`. This is also where Kubernetes (or other container orchestration frameworks) shine.

### Option C: Containerization with Docker and Kubernetes
Going one step further, we enhance our setup with the use of Kubernetes [https://kubernetes.io/]. Kubernetes is one of the better container orchastration systems, two other great options are OpenShift [https://www.openshift.com/] and Mesosphere [https://mesosphere.com/]. I like working with Kubernetes due to the fact that there is a lot of information available online and it is easy to setup.

If you have Docker for Desktop (either Windows or Mac) installed, you can enable Kubernetes in the settings. After a short download, Kubernetes is running locally. It is as easy as that for developers!

Running Kubernetes online is also easy, as it is supported natively (or close to it) by Google, Amazon and Azure.

CI/CD with Kubernetes is also a lot less involved than with (virtual) machines, due to the expressiveness of the `Dockerfile` format and the availability of hosted platforms like the Docker Hub and Github integration, automating the creation of `Docker Images`. As the following image shows, all that is needed after the creation of the Docker Image, is for the Kubernetes service to refresh the pods.

<<< Insert image from >>>

[https://thenewstack.io/ci-cd-with-kubernetes-tools-and-practices/]


## Conclusions
For this assessment, I have evaluated the requirements stated and different options for implementation. Going from option B (Containerization with Docker) into option C (Containerization with Docker and Kubernetes) is a logical advancement and makes filling the automated and interruption-free upgrade requirement less complex.

What I still believe is missing from this solution, is a solid API versioning strategy. In this example, the upgraded version of the airports API brings some breaking changes: using `/search/NL` instead of `/airports/NL`. Some examples of versioning strategies can be found here: https://www.xmatters.com/integrations/blog-four-rest-api-versioning-strategies/. All of the listed options are implementable using HAProxy (our choice of reverse-proxy / load-balancer).