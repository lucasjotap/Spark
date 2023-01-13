This is my main file for keeping the stuff I learned about Apache Spark and its functionalities.

`How did Spark come to be?`

Apache Spark began at UC Berkeley in 2009 as the Spark research project, which was first published the following year in a paper entitled “Spark: Cluster Computing with Working Sets” by Matei Zaharia, Mosharaf Chowdhury, Michael Franklin, Scott Shenker, and Ion Stoica of the UC Berkeley AMPlab. At the time, Hadoop MapReduce was the dominant parallel programming engine for clusters, being the first open source system to tackle data-parallel processing on clusters of thousands of nodes.The AMPlab had worked with multiple early MapReduce users to understand the benefits and drawbacks of this new programming model, and was therefore able to synthesize a list of problems across several use cases and begin designing more general computing platforms.


`What is Spark used for?`

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

Note: We are using two `.option()\`'s and using a csv file format with this example, so keep that in mind.

In order to get the schema information, Spark reads in a little bit of the data and then attempts to parse the types in those rows according to the types available in Spark.You also have the option of strictly specifying a schema when you read in data (which is recommended in production scenarios):

#in Python

**flightData2015 = spark\
.read\
.option("inferSchema", "true")\
.option("header", "true")\
.csv("PATH")**

DataFrames (in Python) have a set of columns with an unspecified number of rows. The reason the number of rows is unspecified is because reading data is a transformation, and is therefore a lazy operation. Spark peeked at only a couple of rows of data to try to guess what types each column should be. To get the schema information, Spark reads in a little bit of the data and then attempts to parse the types in those rows according to the types available in Spark.

![Screenshot from 2022-12-20 14-02-29](https://user-images.githubusercontent.com/98364965/208723916-390ecf92-7cad-43e6-9b1b-c216e53f6996.png)



`Spark Dataframes & SQL?`

So far the descriptions given regarding data writing and reading were simple examples of `transformations`. Spark can run the same `transformations`, regardless of the language, in the exact same way. You can express your business logic in SQL or DataFrames (either in R, Python,Scala, or Java) and Spark will compile that logic down to an underlying plan (that you can see in the explain plan) before actually executing your code. With Spark SQL, you can register any DataFrame as a table or view (a temporary table) and query it using pure SQL. There is no performance difference between writing SQL queries or writing DataFrame code, they both “compile” to the same underlying plan that we specify in DataFrame code.

You can make any DataFrame into a table or view with one simple method call:

*flightData2015.createOrReplaceTempView("PATH")*

With this line of code, you'll have a table or view and now you can query the data using SQL. To do so user the `spark.sql` function (keep in mind, `spark` is our `SparkSession` variable) that conveniently returns a new Dataframe. Although this might seem a bit circular in logic—that a SQL query against a DataFrame returns another DataFrame—it’s actually quite powerful. 

This is a query in SQL to exemplify what it would look like (the data for this query can be found within the files of the learning project.)

#in Python

sqlWay = spark.sql("""
SELECT DEST_COUNTRY_NAME, count(1)
FROM flight_data_2015
GROUP BY DEST_COUNTRY_NAME
""")

dataFrameWay = flightData2015\
.groupBy("DEST_COUNTRY_NAME")\
.count()

With theses line, you'll be shown a Spark `Physical Plan`, this physical plan describes how the framework plans to do the transformation programmed by you.
sqlWay.explain()
dataFrameWay.explain()

Here another example for better visualizing what the `spark.sql` object is capable of doing:

![Screenshot from 2022-12-20 15-52-48](https://user-images.githubusercontent.com/98364965/208744154-ff018f7e-d100-412e-a9da-aa2bc71f3ddd.png)

`Dataframes?`

A DataFrame is the most common Structured API and simply represents a table of
data with rows and columns. The list that defines the columns and the types within those columns is called the schema. You can think of a DataFrame as a spreadsheet with named columns. This is much like a spreadsheet that sits on a single computer but with one fundamental difference, a Spark DataFrame spans across thousands of computers. The reason for such a large operation should be intuitive: either the data is way too large for one computer to handle or it would simply take way too long to perform that computation on one machine.

![Screenshot from 2023-01-13 09-32-16](https://user-images.githubusercontent.com/98364965/212321236-e9936ed6-0e2e-493f-bd09-dd9903d36885.png)

The DataFrame concept is not unique to Spark. R and Python both have similar con‐
cepts. However, Python/R DataFrames (with some exceptions) exist on one machine
rather than multiple machines. This limits what you can do with a given DataFrame
to the resources that exist on that specific machine. However, because Spark has language interfaces for both Python and R, it’s quite easy to convert Pandas (Python) DataFrames to Spark DataFrames, and R DataFrames to Spark DataFrames.

`Partitions?`

To allow every executor to perform work in parallel, Spark breaks up the data into chunks called partitions. A partition is a collection of rows that sit on one physical machine in your cluster. A DataFrame’s partitions represent how the data is physically distributed across the cluster of machines during execution. If you have one partition, Spark will have a parallelism of only one, even if you have thousands of executors. If you have many partitions but only one executor, Spark will still have a parallelism of only one because there is only one computation resource.

An important thing to note is that with DataFrames you do not (for the most part)
manipulate partitions manually or individually. You simply specify high-level transformations of data in the physical partitions, and Spark determines how this work will actually execute on the cluster. Lower-level APIs do exist (via the RDD interface)

`Transformations?`

In Spark, the core data structures are immutable, meaning they cannot be changed
after they’re created. This might seem like a strange concept at first: if you cannot change it, how are you supposed to use it? To “change” a DataFrame, you need to instruct Spark how you would like to modify it to do what you want. These instructions are called transformations.

Transformations are the core of how you express your business logic using Spark. There are two types of transformations: those that specify `narrow dependencies`, and those that specify `wide dependencies`.

Transformations consisting of narrow dependencies (we’ll call them narrow transformations) are those for which each input partition will contribute to only one output partition.

**Narrow dependency**

![Screenshot from 2023-01-13 09-44-17](https://user-images.githubusercontent.com/98364965/212323204-8bf0181f-c7eb-4a00-8ba9-7727993bd9da.png)

A wide dependency (or wide transformation) style transformation will have input
partitions contributing to many output partitions. You will often hear this referred to as a shuffle whereby Spark will exchange partitions across the cluster. With narrow transformations, Spark will automatically perform an operation called pipelining, meaning that if we specify multiple filters on DataFrames, they’ll all be performed in memory. The same cannot be said for shuffles. When we perform a shuffle, Spark writes the results to disk.

**Wide dependency**

![Screenshot from 2023-01-13 09-45-44](https://user-images.githubusercontent.com/98364965/212323455-357324a9-e2f4-417e-9e50-26b8f080cecd.png)

**Now, we'll talk about an important topic**

`Lazy Evaluation`

Lazy evaulation means that Spark will wait until the very last moment to execute the graph of computation instructions. In Spark, instead of modifying the data immediately when you express some operation, you build up a plan of transformations that you would like to apply to your source data. By waiting until the last minute to execute the code, Spark compiles this plan from your raw DataFrame transformations to a streamlined physical plan that will run as efficiently as possible across the cluster. This provides immense benefits because Spark can optimize the entire data flow from end to end. An example of this is something called predicate pushdown on DataFrames. If we build a large Spark job but specify a filter at the end that only requires us to fetch one row from our source data, the most efficient way to execute this is to access the single record that we need. Spark will actually optimize this for us by push‐ing the filter down automatically.


`Spark UI`

Spark also provides a user interface that can be accessed after the it has been started by going to http://localhost:4040 (If you are running in local mode). This will open up a portal interface that you can use to track the work you've done: 

![Screenshot from 2023-01-13 09-53-55](https://user-images.githubusercontent.com/98364965/212324762-038c900f-c63b-407d-837a-e6c74d4b26bd.png)

The Spark UI displays information on the state of your Spark jobs, its environment, and cluster state. And, as you can see on the image above, we're running a PySpark Application. Spark UI is extremely useful for monitoring spark jobs on the cluster.

