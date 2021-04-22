# SumaSoma-Bigdata-Project

## Author : [Suma Soma](www.google.com)

## Text Data Source

- [The Project Gutenberg eBook of The Monk: A Romance, by M. G. Lewis](https://www.gutenberg.org/files/601/601-h/601-h.htm)

## Tools and Languages:

- Python Programming Language
- DataBricks Cloud Platform
- Spark processing engine

## NoteBook Link from Databricks :

[Databricks](https://community.cloud.databricks.com/?o=738325624314186#notebook/123542383545008/command/900976778029719)

## Steps

<br> 
Using urllib.request I'll be pulling data into Notebook and store it in the file named "suma.txt".  I have pulled the data from 'The Monk : A romance' by M.G. Lewis.
<br>
w--------------------------

```
# Reads the data from provided URL and saves it to
import urllib.request
# Open a connection URL using urllib
urllib.request.urlretrieve("https://github.com/Teju2404/tejaswi-bigdata-final-project/blob/main/TheLittleWarrior.txt" , "/tmp/suma.txt")

```

For saving the book , we'll be using dbutils.fs.mv method which takes two arguments which is to send the book from its current location to new location.

```
dbutils.fs.mv("file:/tmp/suma.txt","dbfs:/data/suma.txt")
```

Finally we'll be transferring the file into Spark using RDDs.
We'll be transforming data into Resilient Distributed Databases(RDDs) as Spark holds data in RDDs.

```
sumaRDD = sc.textFile("dbfs:/data/suma.txt")
```

## Cleaning the Data

The data which we took contains sentences,punctuations and stopwords. To clean the data,we split each line by spaces and changing the letters to lower-case and filtering empty lines and breaking sentences into words.

```
wordsRDD=sumaRDD.flatMap(lambda line : line.lower().strip().split(" "))
```

Remove all the punctuations. This can be done using regular expression which looks for anything that is not a letter.
To use Regex , library re needs to be imported.

```
import re
cleanTokensRDD = wordsRDD.map(lambda w: re.sub(r'[^a-zA-Z]','',w))
```

Last task is to remove stopwords. By importing StopWordsRemover PySpark knows what words are stopwords and will filter out the words.

```
from pyspark.ml.feature import StopWordsRemover
remover =StopWordsRemover()
stopwords = remover.getStopWords()
cleanwordRDD=cleanTokensRDD.filter(lambda w: w not in stopwords)
```

## Processing the data

We will map our words into Key-Value pairs('word',1) and then count how many times word occurs.

```
IKVPairsRDD= cleanwordRDD.map(lambda word: (word,1))
```

Reduce by Key : The key is the word in this case. we'll save the word and when it repeats , we will remove it and update count.

```
wordCountRDD = IKVPairsRDD.reduceByKey(lambda acc, value: acc+value)
```

For more complicated processing, we will get back to python to retrieve all elements from data.

```
results = wordCountRDD.collect()
print(results)
```

## Finding Useful Data

- Sorting the words in the descending order and print the results to check the first 13 results in descending order.

```
final_results = wordCountRDD.map(lambda x: (x[1], x[0])).sortByKey(False).take(13)
print(final_results)
```

- To Graph the data we use the library mathplotlib.Using this any type of data can be set to x-axis and y-axis.

```
mostCommon=results[1:5]
word,count = zip(*mostCommon)
import matplotlib.pyplot as plt
fig = plt.figure()
plt.bar(word,count,color="blue")
plt.xlabel("Number of times used")
plt.ylabel("Most used")
plt.title("Most used words in The Monk")
plt.show()
```

# Results

![Sorting](https://github.com/Teju2404/tejaswi-bigdata-final-project/blob/main/sort.PNG)
![Results](https://github.com/Teju2404/tejaswi-bigdata-final-project/blob/main/results.PNG)

# References

- [Matplotlib](https://dzone.com/articles/types-of-matplotlib-in-python)
- [Python](https://www.analyticsvidhya.com/blog/2020/02/beginner-guide-matplotlib-data-visualization-exploration-python/)
- [Dbutils](https://docs.microsoft.com/en-us/azure/databricks/dev-tools/databricks-utils)
