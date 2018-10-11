# DevOps Assessment Documentation

## Background

Provided are two different web API's for a fictional airport application, only GET requests are supported:

* country-service
  * version 1.0.1: a service which returns basic information about countries
    * Endpoint: /countries to get a full list of countries
    * Endpoint: /countries/<query> to search for country by name / ISO code.
* airport-service
  * version 1.0.1: a service which returns information about airports with country codes
    * Endpoint: /airports to get a full list of airports and their runways
    * Endpoint: /airports/<query> to get a list of airports based on country code (e.g.: "NL")
  * version 1.1.0: a service which returns information about airports with country codes
    * Endpoint: /airports?full=[0|1] returns a summary or all details of all airports, depending on the value of full.
    * Endpoint: /airports/<id> returns the full information of an airport based on its identifier. E.g.: /airports/EHAM returns all information for Schiphol.
    * Endpoint: /search/<qry> returns a list of airports based on a country code search.

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

### Option B: Containerization
