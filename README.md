This is my main file for keeping the stuff I learned about Apache Spark and its functionalities.

`How did Spark come to be?`

Apache Spark began at UC Berkeley in 2009 as the Spark research project, which was first published the following year in a paper entitled “Spark: Cluster Computing with Working Sets” by Matei Zaharia, Mosharaf Chowdhury, Michael Franklin, Scott Shenker, and Ion Stoica of the UC Berkeley AMPlab. At the time, Hadoop MapReduce was the dominant parallel programming engine for clusters, being the first open source system to tackle data-parallel processing on clusters of thousands of nodes.The AMPlab had worked with multiple early MapReduce users to understand the benefits and drawbacks of this new programming model, and was therefore able to synthesize a list of problems across several use cases and begin designing more general computing platforms.


`What is Spark?`

Spark has been around for a number of years but continues to gain in popularity and use cases. Many new projects within the Spark ecosystem continue to push the boundaries of what’s possible with the system. For example, a new high-level streaming engine, Structured Streaming, was introduced in 2016. This technology is a hugepart of companies solving massive-scale data challenges, from technology companies like Uber and Netflix using Spark’s streaming and machine learning tools, to institutions like NASA, CERN, and the Broad Institute of MIT and Harvard applying Spark to scientific data analysis.

`Spark’s Basic Architecture`

Single machines do not have enough power and resources to perform computations on huge amounts of information (or the user probably does not have the time to wait for the computation to finish). A cluster, or group, of computers, pools the resources of many machines together, giving us the ability to use all the cumulative resources as if they were a single computer. Now, a group of machines alone is not powerful, you need a framework to coordinate work across them. Spark does just that, managing and coordinating the execution of tasks on data across a cluster of computers. The cluster of machines that Spark will use to execute tasks is managed by a cluster manager like Spark’s standalone cluster manager, YARN, or Mesos. We then submit Spark Applications to these cluster managers, which will grant resources to our application so that we can complete our work.


`Spark Applications`

![Screenshot from 2022-12-16 13-26-18](https://user-images.githubusercontent.com/98364965/208144008-a1c87da3-1107-4df7-9871-c82fc859e944.png)

Spark Applications consist of a `driver process` and a set of `executor processes`. The driver process runs your `main()` function, sits on a node in the cluster, and is responsible for three things: maintaining information about the Spark Application; responding to a user’s program or input; and analyzing, distributing, and scheduling work across the executors. The driver process is absolutely essential— it’s the heart of a Spark Application and maintains all relevant information during the lifetime of the application.

The executors are responsible for actually carrying out the work that the driver
assigns them. This means that each executor is responsible for only two things: executing code assigned to it by the driver, and reporting the state of the computation on that executor back to the driver node.

`How do I write data into Spark?`

Spark includes the ability to read and write from a large number of data sources. To read this data, we will use a `DataFrameReader` that is associated with our `SparkSession`. In doing so, we will specify the file format as well as any options we want to specify. In our case, we want to do something called schema inference, which means that we want Spark to take a best guess at what the schema of our DataFrame should be. We also want to specify that the first row is the header in the file, so we’ll specify that as an option, too.

Note: We are using two `.option()\`'s with this example so keep that in mind.

 in Scala
 
val flightData2015 = spark
.read
.option("inferSchema", "true")
.option("header", "true")
.csv("/data/flight-data/csv/2015-summary.csv")

-----------------

 in Python
 
flightData2015 = spark\
.read\
.option("inferSchema", "true")\
.option("header", "true")\
.csv("/data/flight-data/csv/2015-summary.csv")


`Spark Dataframes & SQL?`





