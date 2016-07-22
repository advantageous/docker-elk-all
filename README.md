# docker-elk-all
Docker ELK stack that is Logback compatible.
(UDP input with JSON codec and elastic search output with JSON codec.)

## Where
This is deployed as [Docker image advantageous/elk](https://hub.docker.com/r/advantageous/elk/)
on docker hub. The source code for this build is at [github](https://github.com/advantageous/docker-elk-all).

## How
The source is config files and a [Packer build project](https://www.packer.io/intro/)
to create a Docker image based on [sebp/elk](https://hub.docker.com/r/sebp/elk/).

## What
The [sebp/elk](https://hub.docker.com/r/sebp/elk/) has excellent
[documentation](http://elk-docker.readthedocs.io/).

## Why
The issue we had with *sebp/elk* docker image was that it was setup for
[Beats](https://www.elastic.co/products/beats)/[Lumberjack](https://github.com/elastic/logstash-forwarder)
log file ingestion and not for use with [Logback](http://logback.qos.ch/) which
sends log data, extra fields and MDC via a JSON codec.

## Adding Logback support UDP and JSON input/output

To support Logback we had to add support for JSON codec output to elastic search.

#### 30-output.conf
```ruby
output {
  elasticsearch {
    hosts => ["localhost"]
    sniffing => true
    codec => json
  }
}
```

We also added support for UDP ingestion.

#### 50-udp.json
```ruby
input {
    udp {
        port => 5001
        codec => json
    }
}
```

## Use

To configure Logback you will need logback and the
[Logback logstash appender](https://github.com/logstash/logstash-logback-encoder).

Then just add the following to a logback.xml config file in your Java project.

#### logback.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <appender name="STASH-UDP" class="net.logstash.logback.appender.LogstashSocketAppender">
        <host>192.168.99.100</host>
        <port>5001</port>
    </appender>


    <root level="INFO">
        <appender-ref ref="STASH-UDP"/>
    </root>

    <logger name="com.mycompany" level="INFO"/>

</configuration>
```

To deploy this with the [advantageous gradle docker plugin](https://github.com/advantageous/docker-test-plugin),
do the following:

#### gradle build.gradle
```java

plugins {
    id "io.advantageous.docker-test" version "0.1.6"
}

...

testDockerContainers {
    elk {
        containerName "elk-app"
        image "advantageous/elk:0.1"
        portMapping(container: 9200, host: 9200)
        portMapping(container: 5044, host: 5044)
        portMapping(container: 5000, host: 5000)
        portMapping(container: 5601, host: 5601)
        portMapping(container: "5001/udp", host: 5001)
        runArgs " /usr/local/bin/start.sh "
    }
}
```

Or run it with docker command line as follows:

```sh
 $ docker run -d -p 9200:9200 -p 5044:5044 -p 5000:5000 -p 5601:5601 \
          -p 5001:5001/udp --name=elk-df advantageous/elk:0.1  \
          /usr/local/bin/start.sh
```

Note that start.sh starts the ELK stack as unix services.

## Build

# [Install packer](https://www.packer.io/intro/getting-started/setup.html).
# Check out the project from [github](https://github.com/advantageous/docker-elk-all).
# Go to the project folder and run packer build as follows.

```sh
$ packer build elk-docker.json
```
