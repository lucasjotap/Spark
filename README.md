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

