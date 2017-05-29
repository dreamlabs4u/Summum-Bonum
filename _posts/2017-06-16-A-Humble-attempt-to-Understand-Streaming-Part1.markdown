---
layout: post
title:  "A Humble attempt to Understand Streaming Data and the way we process it!"
categories: Scala
tags: Scala, Spark, Spark-Streaming
author: Toney Thomas
---

* content
{:toc} 

Recently my friend has asked me, "Dude, it's been quite some time that you have been working on Big Data and related software 
stacks. I have a simple problem and could you please educate me on these jargons and provide a suitable solution ? " 

The inherent repulsive behaviour in me have put me in a defensive mode but I somehow mustered enough courage to hear the problem statement out and 
it goes like this :-

```
"Let's assume that you have a stream of data flowing in and I want to put the data into different buckets by aligning with the following rules :-

    1. The records in a given bucket shouldn't be duplicates 
    2. All the incoming data should be emitted out i.e the collection of data in different buckets will be the same as incoming data. 
    
Once, the data is arranged in different buckets, we should be able to call a Sink. Let's assume this as an HTTP API Service / HDFS output location.     
```

Whenever I am encountered with a problem, I will try to picturise the statement in my mind (I assume our brain has some 'magical' power
to process and give out results when we lift problem statements into diagrams with box and connectors :) ) and that goes like this :-
 
![Problem Statement](/images/sparkstreaming_ps1.png) 

Ok, we are successful in plotting the problem statements using box and connectors. But, I already find myself bombarded with lot of 
questions which I am not certain about.

1. *What is a streaming data ?* 
2. *What is the best mechanism to ingest it.* 
3. *Even if I find a mechanism to ingest the data how much of the incoming data will can be processed at a time ?* 
4. *What all things can be done with the streaming dataset and methods available for sink ?* 

We will try to answer each of these questions at different points in the journey. 

As we all do, my immediate reflex was to google on these jargons and without much hassle I was able to come up with the 
following conclusions.
 
## Batch Data Vs Streaming Data ...

In real world data is generated majorly in two ways, end result of a statistical analysis or by the end result of an event. Lets say,
if I am periodically checking the account balances of customers in a bank or memory usage in a computer and records it as a dataset then 
it can be fairly be categorised as dataset derived out of statistical analysis. There is another class of dataset generated out of instances like
data generated out of sensors, security systems, medical devices based on events happening at their habitat. We couldn't simply predict the occurrence interval of those
data. For instance, lets treat the webserver logs as a dataset and the data will be continuously generated whenever someone accesses any of the
websites hosted on that server. This continuous flow of events can be called as streaming dataset. In a typical Big Data Environment, the need 
for performing real-time stream processing is rapidly increasing and below is some witty but striking example to reason the statement. 

![Strem Vs Batch](/images/streamvsbatch.png)
        
## Indefinite flow of data ...
 
Now that we know the difference between the Batch Data and Streaming data processing, the very first concern that would come to our mind is its
behaviour - the indefinite flow of incoming data. We should have a mechanism to make sure that all the incoming data is processed as and when it 
is available. In layman terms, I can think of couple of ways to handle this scenario 

1. Have a processing layer capable of processing the data as and when it is available. - _#Process_Pattern_1_
2. Have a queueing system in place to hold the incoming data and let the processing layer source the data from there and process. _#Process_Pattern_2_  
   
On further research, I narrowed down my options for the Processing Layer. We can either use Spark Streaming / Storm as a processing layer and 
and Kafka / Flume as queuing system to control the flow of incoming data before it is served for processing.
 
 ![Spark Streaming Architecture](/images/SparkStreamingDiag.png) 

But, in this post we will be concentrating only on the Spark Streaming + Kafka Integration to implement the solution to the problem we had started with.  

## _#Process_Pattern_1_ : Process data as and when it is available. 

Before coming into a solution, we will try to get a basic understanding about the Processing Layer (Spark Streaming) using a simple 
network word count application. Let's assume that we have a netcat server as a source for streaming dataset and our aim is to compute the
word count on the incoming data. In the case of batch mode word count, [Simple Word Count using Spark](), we were reading in the source dataset,
doing the RDD transformations and computes the word count on the whole dataset and finally the results were written on to the disk. In that 
case the actual computation was done on the whole dataset. But, when it comes to the real time streaming, we do not know what can be the size 
of the dataset and when should the computation on that defined dataset be emitted out. This is where the relevance of batch size comes into picture. 

```scala
    // Create the context with a 1 second batch size
    val sparkConf = new SparkConf().setMaster(args(0)).setAppName("NetworkWordCount")
    val ssc = new StreamingContext(sparkConf, Seconds(1))
    
    // Create a socket stream(ReceiverInputDStream) on target ip:port
    val lines = ssc.socketTextStream(args(1), args(2).toInt, StorageLevel.MEMORY_AND_DISK_SER)
    
    // Split words by space to form DStream[String]
    val words = lines.flatMap(_.split(" "))
    
    // count the words to form DStream[(String, Int)]
    val wordCounts = words.map(x => (x, 1)).reduceByKey(_ + _)
    
    // print the word count in the batch
    wordCounts.print()
    
    ssc.start()
    
    ssc.awaitTermination()
 ```
 
Internally, it works as follows. Spark Streaming receives live input data streams and divides the data into batches, which 
are then processed by the Spark engine to generate the final stream of results in batches.


The typical "word count" comes to the rescue. I will try to explain the  
