# A Shiny ✨ AAI Demo FCS Endpoint ✨

The endpoint sources should work out-of-the-box but does not really implement any translation layer for a search engine or performs an actual search. All search requests to CQL will just respond with static results containing some authentication info as demonstration.

_The sources have been generated using the [fcs-endpoint-archetype](https://github.com/clarin-eric/fcs-endpoint-archetype) and cleaned up to only demonstrate the AAI-FCS workflow._


## Deployment files


* [`Dockerfile`](Dockerfile)  
  Multi-stage Maven build and slim Jetty runtime image.
* [`docker-compose.yml`](docker-compose.yml)
* [`pom.xml`](pom.xml)  
  Java dependencies for use with Maven.


## Source code (the interesting files)

* [`eu.clarin.sru.fcs.demo.aai_endpoint.AAIEndpointSearchEngine`](src/main/java/eu/clarin/sru/fcs/demo/aai_endpoint/AAIEndpointSearchEngine.java)  
  The glue between the FCS and our own search engine. It is the actual implementation that handles SRU/FCS explain and search requests. Here, we load and initialize our FCS endpoint.
  It will perform searches with our own search engine (here only with static results), and wrap results into the appropriate output (`eu.clarin.sru.fcs.demo.aai_endpoint.AAIEndpointSRUSearchResultSet`). 
* [`eu.clarin.sru.fcs.demo.aai_endpoint.AAIEndpointSRUSearchResultSet`](src/main/java/eu/clarin/sru/fcs/demo/aai_endpoint/AAIEndpointSRUSearchResultSet.java)  
  FCS Data View output generation. Writes minimal, basic HITS Data View. Here custom output can be generated from the result wrapper `eu.clarin.sru.fcs.demo.aai_endpoint.searcher.MyResults`.
* [`eu.clarin.sru.fcs.demo.aai_endpoint.searcher.MyResults`](src/main/java/eu/clarin/sru/fcs/demo/aai_endpoint/searcher/MyResults.java)  
  Lightweight wrapper around own results that allows access to results counts and result items per index.


## Configuration files

Only the [`log4j2.xml`](src/main/resources/log4j2.xml) is important in case of changing logging settings.

Here are the endpoint configuration:

* [`endpoint-description.xml`](src/main/webapp/WEB-INF/endpoint-description.xml)  
  FCS Endpoint Description, like resources, capabilities etc.
* [`jetty-env.xml`](src/main/webapp/WEB-INF/jetty-env.xml)  
  Jetty environment variable settings.
* [`sru-server-config.xml`](src/main/webapp/WEB-INF/sru-server-config.xml)  
  SRU Endpoint Settings.
* [`web.xml`](src/main/webapp/WEB-INF/web.xml)  
  Java Servlet configuration, SRU/FCS endpoint settings.

The [`demo-aggregator.pem`](src/main/webapp/WEB-INF/demo-aggregator.pem) key is just to demonstrate the format and where to place your key based on the `web.xml` configuration. A valid RSA public key needs to be used to test this feature.


## Build and Run

Build [`fcs.war`](target/fcs.war) file for webapp deployment:

```bash
mvn [clean] package
```

Uses Jetty 10. See [`pom.xml`](pom.xml) --> plugin `jetty-maven-plugin`.

```bash
mvn [package] jetty:run-war
```

NOTE: `jetty:run-war` uses built war file in [`target/`](target/) folder.

Some endpoint/resource configurations are being set using environment variables. See [`jetty-env.xml`](src/main/webapp/WEB-INF/jetty-env.xml) for details. You can set default values there.
For production use, you can set values in the .env file that is then loaded with the `docker-compose.yml` configuration.


The endpoint includes both a [`Dockerfile`](Dockerfile) and a [`docker-compose.yml`](docker-compose.yml) configuration.
The `Dockerfile` can be used to build a simple Jetty image to run the FCS endpoint. It still needs to be configured with port-mappings, environment variables etc. The `docker-compose.yml` file bundles all those runtime configurations to allow easier deployment. You still need to create an `.env` file or set the environment variables if you use the generated code as is.

Using docker:

```bash
# build the image and label it "fcs-endpoint-demo-aai"
docker build -t fcs-endpoint-demo-aai .

# run the image in the foreground (to see logs and interact with it) with environment variables from .env file
docker run --rm -it --name fcs-endpoint-demo-aai -p 8081:8080 --env-file .env fcs-endpoint-demo-aai

# or run in background with automatic restart
docker run -d --restart=unless-stopped --name fcs-endpoint-demo-aai -p 8081:8080 --env-file .env fcs-endpoint-demo-aai
```

Using docker-compose:

```bash
# build
docker-compose build
# run
docker-compose up [-d]
```


## Debugging

Add VSCode default debug setting `Attach by Process ID`, then start the jetty server with the following command, and start debugging in VSCode while it waits to attach.

```bash
# export configuration values, see section #Configuration
MAVEN_OPTS="-Xdebug -Xnoagent -Djava.compiler=NONE -agentlib:jdwp=transport=dt_socket,server=y,address=5005" mvn jetty:run-war
```
