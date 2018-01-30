# Logstash and Elasticsearch

## Introduction

### Description

In [this expirement](https://labex.io/courses/4), we will configure Logstash. Specifically we will configure the input and output endpoint of Logstash because Logstash works as a pipe. Logstash receives log from input endpoint, converts the log format and finally outputs to Elasticsearch. The actual work of log analysis will be done in Elasticsearch.

### What You Will Learn

- Basic configuration of Logstash
- How to start Logstash
- How Logstash connect to Elasticsearch

### Who Is the Intended Audience

This will be the perfect course to start with if you want to learn log analysis or big data. It gives detailed elaborations from installation to basic introduction of how to utilize ELK Stack as well as practical examples for you to practice.

## Principle

We have installed each components of ELKR, but there's still more work to be done. Take a closer look at the following picture. You will find that we will start configuring each component of ELKR stack.

![description of pictures](/upload/N/A/T/rBRXIK5873xb.png)

## Procedures

### Service Start Problem

In the previous experiment, there were some warnings on the screen when we tried to start Logstash and Elasticsearch:

Elasticsearch:
![](/upload/G/K/U/dgrHBNvEr9tu.jpg)

Logstash:
![](/upload/X/A/C/XvDJZti6Nlh7.jpg)

These warnings are caused by the restriction of LabEx's lab environment. To fix it, you just need to comment out some configuration lines.

To fix Elasticsearch's starting warning, open the service configuration file:

```
sudo vim /etc/init.d/elasticsearch
```

Search `/Start Daemon`  keyword in vim, and we can find content as shown in the following screenshot. Comment out all lines: 

```
        #if [ -n "$MAX_OPEN_FILES" ]; then
        #       ulimit -n $MAX_OPEN_FILES
        #fi 

        #if [ -n "$MAX_LOCKED_MEMORY" ]; then
        #       ulimit -l $MAX_LOCKED_MEMORY
        #fi 

        #if [ -n "$MAX_MAP_COUNT" -a -f /proc/sys/vm/max_map_count ]; then
        #       sysctl -q -w vm.max_map_count=$MAX_MAP_COUNT
        #fi 
```
![](/upload/V/R/Y/AF2sVoyJMdCC.jpg)

The warning will disappear after restarting Elasticsearch service.

```
sudo service elasticsearch stop
sudo service elasticsearch start
```

We can also make some configuration lines commented out  in `/etc/init.d/logstash` to fix the problem with Logstash. As shown in below picture:

![](/upload/N/A/C/4rKpMhRn4hMI.jpg)

Restart the Logstash service：

```
sudo service logstash stop
sudo service logstash start
```

### Running Logstash in Command Line

Before configuration starts working, we should check whether all components are in running status:

![](/upload/I/S/D/060GBA9kzf8z.jpg)

Logstash works as a pipe between log files and Elasticsearch. Like other services, Logstash also has its own configuration rules. Configuration of Logstash has three section, including input, filter and output. Before diving deep into the configuration details, we can start Logstash in command line.

In the previous experiment, we have installed Logstash with apt command. The execution command is located in `/opt/logstash/bin` directory. So we use the following command to start Logstash:

```bash
sudo ./logstash -e 'input{stdin{}}output{stdout{codec=>rubydebug}}'
```

> `-e` parameter tells Logstash to use the given string as configuration.

After Logstash has started up, it will wait data from stdin. We can simply type `Hello Logstash!`, then Logstash will receive and convert the data to JSON format. As shown below:

![](/upload/O/P/Q/6Jho4QsdxTTo.jpg)

> You can use  `CRTL + C` key to stop Logstash.

#### Configuration Syntax

Let's first look at the string of the `-e` parameter in the command line above：

```
'input{stdin{}}output{stdout{codec=>rubydebug}}'
```

After format beautification：

```
input {
    stdin {}             
}
output {
    stdout {
        codec => rubydebug
    }
}
```

`stdin {}` tells Logstash to receive data from stdin, while `stdout {}` instructs Logstash to ouput result to stdout.

In the above configuration sample, we actually defined Logstash's two section configurations, `input` and `output`.

In `input` section, we tell Logstash to receive data from stdin. In `output` section，we tell Logstash to ouput the data to stdout, and the `codec` field in `stdout` section instructs Logstash to use `rubydebug` as the format of the result data. 

#### Configure Logstash

We have known the syntax of Logstash configuration, so we can create a configuration file for Logstash. The path of the file is `/etc/logstash/conf.d/logstash-shiper.conf`. Write the following content to `logstash-shiper.conf` file:

```
input {
  stdin {}  
  file {
    path => "/home/labex/Code/elk/access.log"
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
    elasticsearch {
        hosts=>["localhost:9200"]
        index=>"logstash-%{+YYYY.MM.dd}"
    }
}    
```

In the above configuration, Logstash will receive data from stdin and a log file. The file path is `/home/labex/Code/elk/access.log` which is created in the previous lab containing the Nginx access log. Logstash will send data to Elasticsearch server which listens at `localhost:9200` address after finishing data processing.

After the configuration file is created, we can run Logstash with  `-t` parameter to test whether the configuration file is valid：

```
/opt/logstash/bin/logstash -t -f /etc/logstash/conf.d/logstash-shipper.conf
```

If the output is as shown below:

![](/upload/M/H/W/JEa3w4ilyKVa.jpg)


then it means the configuration is valid. Now we can start Logstash with this configuration file:

```
/opt/logstash/bin/logstash -f /etc/logstash/conf.d/logstash-shipper.conf  
```

Now Logstash will read Nginx access log from the `access.log` file, and then send log data to Elasticsearch. Because Logstash is also instructed to send data to stdout, you will find some Nginx log data in console after Logstash has started up.

We should let Logstash run at the background for testing. You can stop Logstash, then use the following command to run Logstash at background.

```
nohup /opt/logstash/bin/logstash -t -f /etc/logstash/conf.d/logstash-shipper.conf &
```

## Summary

In [this experiment](https://labex.io/courses/4) we learned Logstash configuration, enabling it to read access log data from Nginx log file and send the data processed to Elasticsearch. Note that major configurations of Logstash include `input`, `filter` and `output` section.

## Reference

1. [Basic Logstash Example](https://www.elastic.co/guide/en/logstash/current/first-event.html)
2. [Configuring Logstash](https://www.elastic.co/guide/en/logstash/current/configuration.html)


