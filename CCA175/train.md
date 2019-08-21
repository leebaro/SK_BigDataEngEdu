# mysql 접속

mysql 접속
 > mysql -u training -p

database 접속
>  use loudacre;

# sqoop
##  table list 확인
> sqoop list-tables \
--connect jdbc:mysql://localhost/loudacre \
--username training \
--password training

host명은 달라질 수 있다.

## eval을 이용하며 sql 실행하기
> sqoop eval \
--query "SELECT * FROM device limit 5" \
--connect jdbc:mysql://localhost/loudacre \
--username training \
--password training

## eval을 이용하여 테이블 설명 확인
> sqoop eval \
--connect jdbc:mysql://localhost/loudacre \
--username training \
--password training \
--query "desc device"

## import
> sqoop import-all-tables \
--connect jdbc:mysql://localhost/loudacre \
--username training \s
--password training \
--warehouse-dir /loudacre

hdfs에 올라간 파일 확인학
> hdfs dfs -ls /loudacre


## import single table with sqoop
> sqoop import --table accounts \
--connect jdbc:mysql://localhost/loudacre \
--username training \
--password training

hdfs에 올라간 파일 확인하기
> hdfs dfs -ls ./accounts

기존에 올라간 파일 삭제하기
> hdfs dfs -rm -r ./accounts

## import only matching rows from accounts table
> sqoop import --table accounts \
--connect jdbc:mysql://localhost/loudacre \
--username training \
--password training \
--where "state='CA'"

## specifying a file location
> sqoop import --table accounts \
--connect jdbc:mysql://location/loudacre \
--username training \
--password training \
--target-dir /loudacre/customer_accounts


# hdfs
##
hdfs dfs - ls



# RDD



# Spark

> cd $DEVSH/exercises/spark-shell

> pyspark

> sc

## Reading and Displaying a Text File (Python or Spark)
> myrdd = sc.textFile("file:/home/training/training_materials/data/frostroad.txt")

counting the number of lines in the datasets
> myrdd.count()

display the data in the RDD
> myrdd.collect()


## Exploring the Loidacre Web Log Files

create HDFS directory and then copy the dataset from local file system
> hdfs dfs -mkdir /loudacre/

> hdfs dfs -put ~/training_materials/data/weblogs/ /loudacre/

* set a variable for the data files so you do not have to retype the path each time.

> pyspark> logfiles="/loudacre/weblogs/*"  
> pyspark> logsRDD = sc.textFile(logfiles)

* Create an RDD containing only those lines that are requests for JPG Files
> pyspark> jpglogsRDD = logsRDD.filter(lambda line: ".jpg" in line)

* view the first 10 lines of the data
> pypark> jpglogsRDD.take(10)

* Sometimes you do not need to store intermediate objects in a variable,       in which case you can combine the steps into a single line of code. For instance, execute this single command to count the number of JPG requests. (The correct number is 64978.)
> pyspark> sc.textFile(logfiles).filter(lambda line: ".jpg" in line).count()

* Now try using the map function to define a new RDD. Start with a simple map that returns the length of each line in the log file.
> pyspark> logsRDD.map(lambda line: len(line)).take(5)

This prints out an array of five integers corresponding to the first five lines in the file. (The correct result is: 151, 143, 154, 147, 160.)

* That is not very useful. Instead, try mapping to an array of words for each line:
> pyspark> logsRDD.map(lambda line: line.split(' ')).take(5)

* Now that you know how map works, define a new RDD containing just the IP addresses from each line in the log file. (The IP address is the first “word” in each line.)
```
pyspark> ipsRDD = logsRDD.map(lambda line: line.split(' ')[0])
pyspark> ipsRDD.take(5)
```

* Although take and collect are useful ways to look at data in an RDD, their output is not very readable. Fortunately, though, they return arrays, which you can iterate through:
```
pyspark> for ip in ipsRDD.take(10): print ip
```

* save the list of IP addresses as a text files
```
pyspark> ipsRDD.saveAsTextFile("/loudacre/islist")
```

* check the file
```
> hdfs dfs -ls /loudacre/
```

## Bonus exercise


# Hands-On Exercise: Process Data Files with Apache Spark

In this exercise, you will parse a set of activation records in XML format to extract the account numbers and model names.

Important: This exercise depends on a previous exercise: “Access HDFS with the Command Line and Hue.” If you did not complete that exercise, run the course catch- up script and advance to the current exercise:

 ```
 $DEVSH/scripts/catchup.sh
 ```

 ## Reviewing the API Documentation for RDD Operations
* Visit the Spark API page you bookmarked previously. Follow the link for the RDD class and review the list of available operations. (In the Scala API, the link will be near the top of the main window; in Python scroll down to the Core Classes area.)

 ## Reviewing the Data
 * Review the data on the local Linux filesystem in the directory $DEVDATA/activations. Each XML file contains data for all the devices activated by customers during a specific month
```
<activations>    
  <activation timestamp="1225499258" type="phsone">
    <account-number>316</account-number>
    <device-id> d61b6971-33e1-42f0-bb15-aa2ae3cd8680 </device-id> <phone-number>5108307062</phone-number> <model>iFruit 1</model>
  </activation> …
</activations>
```

 * Copy the entire activations directory to /loudacre in HDFS.
 ```
 hdfs dfs -mkdir /loudacre/
 hdfs dfs -put $DEVDATA/activations /loudacre/
 ```

 ## Processing the files
Follow the steps below to write code to go through a set of activation XML files and extract the account number and device model for each activation, and save the list to a file as account_number:model.
*   Start with the ActivationModels stub script in the exercise directory: $DEVSH/exercises/spark-etl. (A stub is provided for Scala and Python; use whicheverS language you prefer.) Note that for   convenience you have been provided with functions to parse the XML, as that is not the focus of this exercise. Copy the stub code into the Spark shell of your choice.

```
import xml.etree.ElementTree as ElementTree

# Optional: Set logging level to WARN to reduce distracting info messages
sc.setLogLevel("WARN")

# Given a string containing XML, parse the string, and
# return an iterator of activation XML records (Elements) contained in the string

def getActivations(s):
    filetree = ElementTree.fromstring(s)
    return filetree.getiterator('activation')

# Given an activation record (XML Element), return the model name
def getModel(activation):
    return activation.find('model').text

# Given an activation record (XML Element), return the account number
def getAccount(activation):
    return activation.find('account-number').text

```

* Use wholeTextFiles to create an RDD from the activations dataset. The resulting RDD will consist of tuples, in which the first value is the name of the file, and the second value is the contents of the file (XML) as a string.

```
pyspark> activationfiles="/loudacre/activations/*"
pyspark> activationsRDD = sc.wholeTextFiles(activationfiles)
```

* Each XML file can contain many activation records; use flatMap to map the contents of each file to a collection of XML records by calling the provided getActivations function. getActivations takes an XML string, parses it, and returns a collection of XML records; flatMap maps each record to a separate RDD element.

```
activationRecords = activationsRDD.\
flatMap(lambda (filename,xmlstring) : getActivations(xmlstring))
```

*  Map each activation record to a string in the format accountnumber:model. Use the provided getAccount and getModel functions to find the values from the activation record.

 ```
 models = activationRecords. \
 map(lambda record: getAccount(record) + ":"+getModel(record))
 ```


* Save the formatted strings to a text file in the directory /loudacre/account-models.

```
models.saveAsTextFile("/loudacre/account-models")
```

# Hands-On Exercise: Use Pair RDDs to Join Two Datasets
