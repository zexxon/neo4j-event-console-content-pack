Neo4j Demo Content Pack
=======================

Graylog Content Pack which demonstrates advanced log management use cases with
Neo4j graph database integration

-   Time Buckets Relationships

-   Key Relationships

-   Dependency Relationships

-   Basic Non-Interactive Event Console with Log Aggregation

###### Requires Neo4j output plugin which can be found [here](<https://marketplace.graylog.org/addons/06bf2e54-1174-4480-9ec6-0e15e3413baf>)

##### **Note:** Ensure you create a unique constraints on all nodes otherwise you may see duplicate nodes (due to the multithreaded nature of Neo4j).

**Graylog & Neo4j Setup**

For a quick-start Neo4j can be started as a single instance. S

imply download the Neo4j community edition suitable for your operating system.

As an example we install and start Neo4j on a Ubuntu system:

**Install Java 8**

`$ sudo add-apt-repository ppa:webupd8team/java`

`$ sudo apt-get update`

`$ sudo apt-get install oracle-java8-installer`

**Download and start Neo4**j

`$ wget -O neo4j-community-2.3.1-unix.tar.gz \
http://neo4j.com/artifact.php?name=neo4j-community-2.3.1-unix.tar.gz`

`$ tar xzvf neo4j-community-2.3.1-unix.tar.gz`

`$ cd neo4j-community-2.3.1/ $ ./bin/neo4j console`

Now you have a Neo4j instance running in the foreground.

This is fine for testing, for a production setup please consider reading more
about server installation scenarios
[here](<http://neo4j.com/docs/stable/server-installation.html>).

To make the server listen on all network interfaces edit
`conf/neo4j-server.properties` and set `org.neo4j.server.webserver.address` to
`0.0.0.0`.

 

The next step is to install the Neo4J output plugin on your Graylog instance. If
you are using one of our pre-configured appliances copy the plugin to
`/opt/graylog/plugin`

`$ wget
https://github.com/mariussturm/graylog-plugin-output-neo4j/releases/download/1.0.0/plugin-output-neo4j-1.0.0.jar
$ sudo cp plugin-output-neo4j-1.0.0.jar /opt/graylog/plugin $ sudo graylog-ctl
restart graylog-server`

A simple example To test the setup we can start with a simple graph. Graylog
ships with a build-in random log generate that simulates a HTTP server. To
visualize the incoming connections of this fictitious web server we first create
the input:

Only messages coming from this input should be represented in the graph. To
identify each connection we create a relationships between the `user_id` of a
connection and the HTTP server `example.org`. To filter the right messages we
create a new stream with the following rules:

Now the Neo4j output can be attached to the stream. The default Cypher query is
already made for this example and can be adopted. Only the URL to the Neo4j
server and the username and password needs to be changed for the local
occurrence (default user/pass: neo4j/neo4j).

A look into the built-in Neo4j web interface shows the results. A query like
`MATCH (n) RETURN n;` returns all `user_id` connected to the test web server.

![](<images/simple-dependency-chart.png>)

Advanced Demo
-------------

You will need an updated Random HTTP Message Generator input , which is
available in the Graylog master branch and will be going into the 2.0 release.

-   Make sure you have the Neo4j output plugin installed

-   Download and install the demo content pack

-   Configure the Neo4j Host/IP and credentials on the output source

 

Once you have installed the content pack and updated your Neo4j output
configuration, you should start seeing messages like the following appear in
your Graylog instance:

![](<images/new-random-http-message-generator.png>)

In Neo4j you could see something like the following:

![](<images/time-bucket-graph.png>)

Now we can use a simpler query in the output plugin, based on dependencies only:

![](<images/dependency-graph.png>)

 

### Simple read only event console within Neo4j

![](<images/neo4j-event-console.png>)

![](<images/neo4j-event-console-2.png>)

The above console was built using the following Cypher query in Graylog.

###### **Note:** The Cypher query below is dependent on a newer version of the RandomHTTPMessageGenerator input plugin. which will be included in the upcoming Graylog 2.0 release.

`MERGE (dependency:DEPENDANT { dependency: ‘${dependency}’}) ON CREATE SET
dependency.firstSeen = ‘${ingest_time}’, dependency.firstSeenepoch =
‘${ingest_time_epoch}’, dependency.count = 1`

`ON MATCH SET dependency.lastSeen = ‘${ingest_time}’, dependency.lastSeenepoch =
‘${ingest_time_epoch}’ ,dependency.count = dependency.count + 1`

`MERGE (source:HOST { address: ‘${source}’})\nON CREATE SET source.firstSeen =
‘${ingest_time}’, source.firstSeenepoch = ${ingest_time_epoch}’, source.count =
1, source.last_response_code = ‘${http_response_code}’`

`ON MATCH SET source.lastSeen = ‘${ingest_time}’, source.lastSeenepoch =
‘${ingest_time_epoch}’,source.count = source.count + 1,
source.last_response_code = ‘${http_response_code}’`

`MERGE (minute:MINUTE { time: ‘${minute}’})`

`ON CREATE SET minute.firstSeen = ‘${ingest_time}’, minute.firstSeenepoch =
‘${ingest_time_epoch}’,minute.count = 1`

`ON MATCH SET minute.lastSeen = ‘${ingest_time}’,minute.lastSeenepoch =
‘${ingest_time_epoch}’, minute.count = minute.count + 1`

`MERGE (source)-[:TIME]->(minute)`

`CREATE UNIQUE (dependency)-[:TIME]->(minute) CREATE UNIQUE
(source)-[:DEPENDS]->(dependency)`
