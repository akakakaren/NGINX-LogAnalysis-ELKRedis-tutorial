# Build Nginx Log Analysis Sytem Based on ELKR - content overview see below 
how to build ELKR, a complete set of Nginx log analysis tool
* ### Installation
* ### Logstash and Elasticsearch
* ### Data visualization based on Kibana
* ### Build distributed ELK with Redis

This project tutorial is located on [LabEx](https://labex.io/courses/4), and you can also use the [virtual environment](https://labex.io/courses/4) to practice this tutorial, which is highly recommended. Should be very helpful. 

## Lab 1 - Installation
## Introduction

### ELKR Description

ELKR (Elasticsearch + Logstash + Kibana + Redis) is a popular data analysis tool stack. It makes searching and analyzing data easier than ever before. In this course, we will learn the basic concepts and usage of ELKR to analyze Nginx log. What you need to do in this first experiment is install each component of ELKR.  If you encounter any problem while you're learning, stop by our discussion forum and ask a question here [LabEx forum](https://labex.io/questions) or refer back to official documents.

###  What You Will Learn

- Understand the basic concepts of ELK Stack
- How to install each component of ELK

### Who Is the Intended Audience

This will be the perfect course to start with if you want to learn log analysis or big data. It gives detailed elaborations from installation to basic introduction of how to utilize ELK Stack as well as examples for you to practice. 

##  ELK Overview

ELK(R) = Elasticsearch + Logstash + Kibana (+ Redis) is a data analysis and visualization tool stack.

* Elasticsearch is the key essence of the entire stack system. It is responsible for storing, indexing, and analyzing data. We will store all Nginx logs in Elasticsearch.

* Logstash is a data processing pipeline. It can ingest data from multiple sources simultaneously, transform data, and then send the data to Elasticsearch. We can use Logstash to send Nginx logs from multiple web servers to a single Elasticsearch.
 
* Kibana is a front-end web tool designed to make Elasticsearch better interact with users and it has powerful data visualization capabilities.

* Redis itself is not part of the ELK stack but exists as a powerful plugin to the ELK stack. Redis works as a broker between data source and Logstash. For example, web server does not need to know anything about Logstash, and it just simply sends data to Redis. Similarly，Logstash only need to read data from Redis. Redis decouples data source and Logstash.

The architecture of ELKR


![image desc](/upload/I/S/R/3ge4urnx9msN.png)


## Procedures

### Installation
#### Check System Environment

Before installing ELK(R)，you should first check the system environment. Open Xfce terminal in our virtual environment and use the following command:

```bash
cat /etc/issue
```

to get the operating system version, as shown below:

![](/upload/Q/E/K/9danvzdOwYcX.jpg)


> the operating system version in LabEx lab is Ubuntu 14.04

Second, Elasticsearch is a Java program language project，therefore you should make sure that Java runtime has been installed. Use the command:

```bash
java -version
```

to check if Java has been installed, as shown below:

![](/upload/O/A/V/fbnzzAODjeCJ.jpg)

> Java runtime has been installed in LabEx lab environment.

As we said before，Redis is a powerful plugin for Logstash. You can use the following command to check if Redis has been installed.

```bash
redis-server -v
```

The command execution result is shown below：

![](/upload/T/K/W/UUu5Qoi48I6B.jpg)

> Redis server has been installed in LabEx lab environment.

We can use the command below to check if Redis server is running：

```bash
netstat -antulp | grep 6379
```

If Redis server is not running, you can use the command below to start Redis server.

```bash
sudo service redis-server start
```

As shown below:

![](/upload/B/D/C/DbzkiKNkH5SY.jpg)

>  Sometimes you cannot stop Redis server with command `sudo service stop  redis-server`.  To solve this, use `kill` command to force Redis server to stop. 

> ```

> ps -u labex -o pid,rss,command | grep redis
Then

> kill -9 the_pid
```

#### Install Elasticsearch

We can install Elasticsearch in different ways like from source code or through DEB package. But `apt` package manager is the best choice to install Elasticksearch.

First, we need to add apt key：

```bash
wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```

Then add official apt source to enable repository

```bash
echo "deb https://packages.elastic.co/elasticsearch/2.x/debian stable main" | sudo tee -a /etc/apt/sources.list.d/elasticsearch-2.x.list
```

![](/upload/W/P/I/WmbkZ6pKHIoV.jpg)

Run apt-get update

```bash
sudo apt-get update
```

Now, you can install it with：

```bash
sudo apt-get install elasticsearch
```

![](/upload/N/Y/D/gJpZhm2Trwrv.jpg)

Wait until installation command running has finished, and then we can start the Elasticsearch service：

```bash
sudo service elasticsearch start
```

Elasticsearch service's starting process might fail because of LabEx's lab environment limitation. We will fix this problem in the next Lab.

![](/upload/O/V/B/y7CMgfl2brYq.jpg)

####  Install Logstash

We can also use `apt` command to install Logstash.

Add key:

```bash
echo "deb https://packages.elastic.co/logstash/2.3/debian stable main" | sudo tee -a /etc/apt/sources.list
```

Update the repository and install Logstash:

```bash
sudo apt-get update
sudo apt-get install logstash
```

![](/upload/M/I/V/uGzDNk0zHZI6.jpg)

Start Logstash service：

```bash
sudo service logstash start
```

Same with Elasticsearch Lostash. The starting process may fail because of LabEx's lab environment limit. We will fix this problem in the next Lab.

![](/upload/C/E/K/rniiRf86ANti.jpg)


####  Install Kibana

We can also use `apt` command to install Logstash.

Add apt key:

```bash
echo "deb http://packages.elastic.co/kibana/4.5/debian stable main" | sudo tee -a /etc/apt/sources.list
```

![](/upload/M/B/N/ceMlMlj3K4vW.jpg)

Update the repository and install Kibana:

```bash
sudo apt-get update
sudo apt-get install kibana
```

![](/upload/K/O/I/4kRkE9EHNyfa.jpg)

Start Kibana:

```bash
sudo service kibana start
```

![](/upload/W/V/H/cqQE972bpjKC.jpg)

Visit `http://localhost:5601` in your browser. You can see that Kibana is running.

![](/upload/D/W/V/B1n1erwWCrn4.jpg)


### Generate Nginx Log

Let's continue installing Nginx to generate access log for analysis. We can also use apt to install Nginx. Becasue LabEx's lab envirionment has already installed Nginx, you just need to start it.

```
sudo service nginx start
```

After Nginx has started, access `http://localhost` in browser.

![](/upload/C/S/W/9ABwcnPDeniX.jpg)

We need to configure `nginx` to generate access log. Add the following line to the default configuration file of Nginx which is located in `/etc/nginx/sites-available/default`:

```
access_log /home/labex/Code/elk/access.log;
```

The configuration line above will tell Nginx to generate access log and write it to  `/home/labex/Code/elk/access.log` file. So please make sure `/home/labex/Code/elk` directory as already existed and Nginx has permission to create files in that directory.

![](/upload/C/O/N/mrSqS0wMG6c2.jpg)


Restart Nginx service because the configuration has changed.

```
sudo service nginx restart
```

Access `http://localhost/` in your browser, and the accessed log will be generated.

![](/upload/J/L/Q/J4C1Gr4VIAIz.jpg)

## Summary

You can install all ELK components through source package as an alternative. But APT package manager is the best choice, because apt command enables us to use `service` command to start, stop and check service status.

```bash
# check service status
sudo service redis-server status
sudo service elasticsearch status
sudo service logstash status
sudo service kibana status
```
In the [next experiment](https://labex.io/courses/4), we are going to build ELK, as well as make log transfer to Elasticsearch through Logstash.

## Reference

1. [Elasticsearch Setup](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-repositories.html)
2. [Logstash Setup](https://www.elastic.co/guide/en/logstash/current/installing-logstash.html)
3. [Kibana Setup](https://www.elastic.co/guide/en/kibana/current/setup.html)

Also learn other tutorials [here](https://labex.io/courses), same platform.
