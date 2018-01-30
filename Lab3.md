
# Data Visualization Based on Kibana

## Introduction

### Description

In the [previous experiment](https://labex.io/courses/4), we have successfully sent log data to Elasticsearch from Nginx log file based on Logstash. In this experiment, we will continue to configure ELK stack so that we can explore and visualize the log data on Kibana dashboard. 

###  What You Will Learn

- Basic Use of Kibana
- Basic Query of Lucene

## Procedures

###  Basic Usage

In the previous experiment, we have run Kibana service in the background. Open `http://localhost:5601` in your browser, you will see the Kibana dashboard. Kibana prompts us to create the index for Elasticsearch data. We can use default timestamp index mode. 

![](/upload/E/C/V/5pik9BRETycn.jpg)

When you click the `Create` button, Kibana will show 18 log fields for us:

![description of pictures](/upload/A/L/W/f6eNIG5kB96u.png)

When click `Discover` menu at the top of the page, Kibana will show the Elasticsearch data, including a search input field.

![](/upload/W/U/E/tfSnIVbVsqU0.jpg)


The `Visualize` menu will show us the excellent data visualization interface.

### Basic Statistics Example

Suppose we want to calculate how many Nginx access logs are generated per minute. We can click `Line chart` and then click `From a New search`, shown as below：

![description of pictures](/upload/K/A/C/j4bCGJdl93Y5.png)


In the right side of the interface, `Metrics` is responsible for a eigenvalue, where we can see the Y-axis is Counter, that is, to calculate the data.

> If your data is not big enough, you can insert some data to access.log manually.

We also need to tell Kibana what we want to caculate, so we need to set `Buckets`. Click X-axis to create Aggregation on the X-axis. The configuration is shown below：

![description of pictures](/upload/R/J/O/s1kUks5DM2tr.png)

The Nginx log sent to Elasticsearch in the past 15 minutes is shown at the right side of the figure.

### Save Visualization Template

Kibana can save configuration template so that you can quickly see the most recent visualization result. You can save current configuration by clicking `Save` button in the top right corner:

![description of pictures](/upload/J/U/A/BLHzym3P1Flk.png)


After clicking `Save` button, Kibana need you to input a template name. We can input `Log-Count` as the name for saved configuration.

Now, when restarting the browser and clicking the `Visualize` Menu, you will see the saved visualization tempates.

![description of pictures](/upload/O/K/M/VQMgbIxCkw6i.png)


Similarly, if you have many visualization configurations, you can save all of them to the top `Dashboard`

If you want the interface to look cool, you can switch to Kibana's Dark theme in the settings：

![description of pictures](/upload/X/B/T/Oz5H7sKlpvv9.png)

The Dashboard becomes fabulous when you have various visualization templates, for example：

![description of pictures](/upload/S/X/J/oZqkzqnhPSqq.png)


For more use cases, you can query the official user manual：[https://www.elastic.co/guide/en/kibana/current/index.html](https://www.elastic.co/guide/en/kibana/current/index.html)


###  Advanced Query

If you need a more accurate query in Kibana, you should use Lucene which is the core of Elasticsearch.

For example, let's try searching string in Elasticsearch data field.

First of all, we should fake the Elasticsearch data. Run Logstash with command  `sudo /opt/logstash/bin/logstash -f /etc/logstash/conf.d/logstash-shipper.conf ` in terminal, and then enter `hello world`、`how do you do`、`what's wrong with you` to terminal. When Logstash receives these strings, it will send to stdout and Elasticsearch, as shown below:


![](/upload/L/G/V/t6LJ0hq3sRoD.jpg)


Set Kibana to auto refresh every 5 seconds:

![](/upload/J/E/F/nKsqnRCmT6ZA.jpg)

Now we can see that the data we enter into terminal has been displayed on Kibana page：

![](/upload/A/L/A/5M3pFvrEPLGB.jpg)


Next, we are going to search all records with the word `you`. Here, we can input ：`message: *you*`. The `message` here is the field we want to search. `you*` is the value of `message` field.  The entire pattern means that Kibana should search any record whose `message` field contains string `you`.

After you entered search pattern and clicked the search icon, you will see the results matched.  `what's wrong with you` and `how do you do` string, those we entered before and matched the search pattern have been  highlighted. Shown below:

![](/upload/I/Q/D/5U6Pbti5u3nN.jpg)

> Search syntax shown above does not belong to Kibana but is owned by Lucene which is the core of Elasticsearch. Kibana only serves as a front-end for Elasticsearch.
> 
> For more details about Lucene search syntax, you can refer to：[Lucene Query Analyzer syntax](https://lucene.apache.org/core/2_9_4/queryparsersyntax.html)

##Summary

In [this lab](https://labex.io/courses/4), we experiment with the visualization function of Kibana and get a glimpse of the Lucene query syntax needed in Elasticsearch. We will establish a distributed ELK stack based on Redis in the [next lesson](https://labex.io/courses/4).

#### Further Reference for Reading

1. [Kibana guide for users](https://www.elastic.co/guide/en/kibana/current/index.html)
2. [Lucene Query Analyzer syntax](https://lucene.apache.org/core/2_9_4/queryparsersyntax.html)


