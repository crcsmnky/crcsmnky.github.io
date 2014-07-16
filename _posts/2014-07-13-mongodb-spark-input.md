---
layout: post
title: MongoDB & Spark - Input
comments: true
---

Say your writing a [Spark](http://spark.apache.org) application and you want to pull in data from [MongoDB](http://www.mongodb.org). There are a couple of ways to accomplish this task.

### Directly from MongoDB

To read directly from MongoDB, create a new `org.apache.hadoop.conf.Configuration` with (at least) the parameter `mongo.job.input.format` (set to `MongoInputFormat`). Then use your [SparkContext](http://spark.apache.org/docs/latest/programming-guide.html#initializing-spark) to create a new RDD from a Hadoop-backed file using the `newAPIHadoopFile(...)` method:

{% highlight java %}
Configuration inputDataConfig = new Configuration();
inputDataConfig.set("mongo.job.input.format",
    "MongoInputFormat.class");
JavaPairRDD<Object,BSONObject> inputData = sc.newAPIHadoopFile(
    "mongodb://127.0.0.1:27017/test.foo", MongoInputFormat.class,
    Object.class, BSONObject.class, inputDataConfig);
{% endhighlight %}

The first argument to `newAPIHadoopFile` is the path to the data to be read. This should be a valid [MongoDB connection string](http://docs.mongodb.org/manual/reference/connection-string/) including the database and collection name (e.g. "test.foo").

### Reading BSON Files

This step assumes you have already used [mongodump](http://docs.mongodb.org/manual/reference/program/mongodump/) to dump the contents of your database as a series of BSON files, one per collection, and stored them somewhere accessible (S3, HDFS, etc.).

Like before, create a new `org.apache.hadoop.conf.Configuration` with `mongo.job.input.format` set to `BSONFileInputFormat`. You'll use the same function `newAPIHadoopFile(...)` as before but this time the first argument should be the full path to your BSON file. You'll need to read each one individually into it's own RDD.

{% highlight java %}
Configuration bsonDataConfig = new Configuration();
bsonDataConfig.set("mongo.job.input.format",
    "BSONFileInputFormat.class");
JavaPairRDD<Object,BSONObject> bsonData = sc.newAPIHadoopFile(
    "hdfs://namenode:9000/data/test/foo.bson",
    BSONFileInputFormat.class, Object.class,
    BSONObject.class, bsonDataConfig);
{% endhighlight %}

*Update*: here's a link to my [mongodb-spark-demo](https://github.com/crcsmnky/mongodb-spark-demo) repo if you want to see an example in action.