
# Build Distributed ELK with Redis

## Introduction

### Description

This section uses two logstash. One is to input to redis，and the other one is to output from redis，realizing the prototype of distributed redis storage log. Hopefully you can have a taste of the principle and really comprehend it.

### What You Will Learn

- Logstash plugin
- Redis environment deployment

## Principle

In the second lab, we have simplified the data flow of ELK to `Log-> Logstash-> Elasticsearch-> Kibana`. Now we will add Redis to the ELK stack and change the data flow to the figure below:

![description of pictures](/upload/B/K/B/wmh5MiqOJNdH.png)


## Procedures

### Configure Redis as Logstash Plugin

Redis itself is not part of the ELK stack but is a powerful plugin to ELK stack.  Redis works as a broker between data source and Logstash. For example, web server does not need to know anything about Logstash, and it just sends data to Redis. Similarly，Logstash only needs to read data from Redis. Redis decouples data source and Logstash.

Fortunately, the Redis plugin has been integrated into the latest version of Logstash. We only have to configure Logstash to enable Redis. The configuration to enable Redis plugin for Logstash shipper is as shown below：

```
input {
  stdin { }
  file {
    path => "/home/shiyanlou/Code/elk/access.log"
    start_position => beginning
    codec =>  multiline {
      'negate' => true
      'pattern' => '^\d'
      'what' => 'previous'
    }
  }
}
output {
  stdout {
    codec => rubydebug
  }
  redis {
    host => "localhost"
    data_type => "list"
    key => "logstash"
  }
} 
```

We instruct Logstash to send data to key `logstash` whose type is `list` in Redis server which listens at `localhost`.

### Configure Logstash Indexer

We need to create a new configuration file for new Logstash process working as an indexer, as shown below:

```
input { 
  redis {
    host => "localhost"
    type => "redis"
    data_type => "list"
    key => "logstash"
  }
}
output { 
  stdout {
    codec => rubydebug
  } 
  elasticsearch {
    hosts=>["localhost:9200"]
    index=>"logstash-%{+YYYY.MM.dd}"
  }
}
```

The above configuration tells Logstash to receive data from Redis's key `logstash` and sends data to stdout and Elasticsearch.

Then we run two Logstash processes with separate configuration file above. When Logstash shipper produces new data（when we enter string `hello world`), you will find that the indexer will also generate new data:

![](/upload/E/A/I/JHoQHb8czgd0.jpg)

## Summary

In this lab, we build a local distributed ELK stack based on Redis. When two Logstash are running on different servers with these two configuration files, you can have a real distributed ELK stack. The only thing to modify is the Redis host address in the configuration.

## Reference

1. [Logstash Redis configuration reference](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-redis.html)


