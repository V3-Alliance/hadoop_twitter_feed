Hadoop Twitter feed analysis
============================

This repo contains the flume and hive scripts that are used to grab the streamed data from the twitter stream api and then clean/structuring the raw feed to data tables that can be easily queried. This project is meant for a testing/experimentation with hadoop and all the tools and apps that come with it. It is not meant to be used in a production environment but the scripts can be easily tweaked if necessary to be deployed in a production setting.

Tools used/required
-------------------

**Hadoop**
Tested on Cloudera virtual machine image running of VirtualBox.

**Flume**
Using flume-twitter.conf to download live twitter stream and save it directly to hadoops hdfs.

**Hive**
Using hive script to structure the streamed results into data tables, cleaning the data and using the provided dictionary dataset to get the sentiment view of a twitter post by comparing words from the tweet to the dictionary dataset to get the polarity rating.

Setting up Hadoop
-----------------

The quickest way to setup hadoop is to download the Cloudera quickstart vm image, you can get the image from their [website](http://www.cloudera.com) or if you just google "Cloudera VM image". Cloudera provides multiple vm images for various vm platforms, VirtualBox was the image used for this project but you can choose any of the other images Cloudera has provided.

1. To use the VirtualBox image, first download the latest version of [VirtualBox](http://www.virtualbox.org/) and install it on your machine.

2. Download the Cloudera VirtualBox image from their [website](http://www.cloudera.com/content/cloudera/en/downloads/quickstart_vms.html), the download size will be about 4gb.

3. Run VirtualBox on your machine, then go to "File" and select "New". It will ask you for the vm image file you just downloaded, select the image and follow the instructions. You will need to assign the vm at least 4gb of memory, so make sure your host machine has enough system memory to allocate to the vm. The initial process will take awhile but once it is complete start the vm and you will be done.

Using Flume
-----------

Flume will be used to download the twitter stream directly into hadood HDFS. The stream downloaded from twitter will be in JSON format. You will need custom flume jar that will handle the connection to twitter and and ingest the json into HDFS, you can download a pre-compiled jar or build it yourself from [here](https://github.com/cloudera/cdh-twitter-example). To build your own jar clone the repo and from there: 

<pre>
$ cd flume-sources
$ mvn package
$ cd ..
</pre>

1. Once you've acquired your "flume-sources-1.0-SNAPSHOT.jar" file, you need to copy that file to "/usr/lib/flume-ge/lib".

2. Modify the "flume-twitter.conf", be sure to include your twitter consumerKey, consumerSecret, accessToken and accessTokenSecret. If you don't have your creditentials from twitter you need to register for a twitter account and dev app to get your twitter creditentials to access their streaming api. Set the keywords for the tweets you want to capture(e.g: iphone6, iphone 6, iphone6plus..), the HDFS location to save the streamed data to and the transaction capacity sizes to meet your configuration. (You can leave these to the default values if your using the Cloudera vm image)

3. To run the stream:

<pre>
$ flume-ng agent -f <path_to_conf>/flume-twitter.conf -Dflume.root.logger=DEBUG,console -n TwitterAgent
</pre>

Use "Ctrl + c" to stop the stream. 


Using Hive to store streamed data to datatables in hadoop
---------------------------------------------------------

The Hive script in this repo is used to move the json data to a structured datatable, clean it and process the sentimental value of a tweet by evaluating the words in a tweet to determine if the tweet is a positive, negative or neutral tweet. You'll need to install 'hive-serdes' jar that will serialize and deserialize json. You can download it or build it yourself from [here](https://github.com/cloudera/cdh-twitter-example). To build your own jar clone the repo and from there: 

<pre>
$ cd hive-serdes    
$ mvn package  
$ cd ..
</pre>

1. Once you 've acquired your "hive-serdes-1.0-SNAPSHOT.jar" file, you need to copy the file to "/usr/lib/hive/lib".

2. You need to upload this repos "data" folder with all its contents to the HDFS.

3. Modify the "hiveddl.sql" script, change the "location" to point to the directory to where your streamed twitter data is saved to and where the "data" folder you uploaded to the HDFS.

4. Run the hive script:

<pre>
$ hive -f hiveddl.sql
</pre>

5. Once its finished it should create a bunch of datatables that you can query using sql in hive, there are 2 tables that should be of interest: tweets_sentiment which gives each tweet a sentiment rating of positive, negative or neutral and tweetsbi which contains the sentiment value, date of tweet and the country of the tweeter.

6. You can then using the available data you can export them to xls and create your own charts and graphs or you can continue to refine the data using hive and sql.

