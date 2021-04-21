# suma-bigdata-project
This project is based on displaying the word count in a file using PySpark and Databricks in cloud.
# Author: **[Suma Soma]()**
# Tools and Languages:
### Tools: 
Pyspark, Databricks Notebook, Pandas, MatPlotLib, Seaborn 
### Language: 
Python
**Important terms to know**: 
- SparkContext(sc) - entry point to any spark functionality.
- Resilient Distributed Dataset (RDD) - A distributed collection of Objects
## Input Source is taken from:
https://www.gutenberg.org/files/65126/65126-0.txt
# Process and steps:
### step 1:
The first and most important move is to pull the data using the urllib.request library. The information retrieved is saved in a temporary tab. We're storing this information in a temporary file called sumasoma.txt. I'm using the following source file/text data: An Eel by the Tail is available as a Project Gutenberg eBook.
```
import urllib.request
urllib.request.urlretrieve("https://github.com/suma-gitrep/suma-bigdata-project/blob/main/An%20Eel%20by%20the%20Tail.txt" , "/tmp/sumasoma.txt")
```
### step 2:
We'll use dbutils.fs.mv to transfer the data to a new place now that it's been processed. Two arguments are used in the dbutils.fs.mv program. The first parameter specifies the text file to be transferred, and the second specifies where the file should be moved.
```
dbutils.fs.mv("file:/tmp/sumasoma.txt","dbfs:/data/sumasoma.txt")
```
### step 3:
The next move is to use sc.textfile to import the data into Spark.
```
sumasomaRDD = sc.textFile("dbfs:/data/sumasoma.txt")
```
## Cleaning the data:
### step 4:
Capitalization, punctuation, phrases, and stopwords are actually present in the ebook. The first step in determining the word count is to flatmap and remove capitalization and spaces.
```
# flatmap each line to words
suma_wordsRDD = sumasomaRDD.flatMap(lambda line : line.lower().strip().split(" "))
```
### step 5:
The punctuation is then removed. We're going to use the re library to get rid of any characters that don't look like letters.
```
import re
# remove punctutation
cleaned_tokens_RDD = suma_wordsRDD.map(lambda w: re.sub(r'[^a-zA-Z]','',w))
```
### step 6:
The next step will be to delete any stopwords that remain after the punctuations have been deleted. The StopWordsRemover library from pyspark.ml.feature is used to accomplish this.
```
from pyspark.ml.feature import StopWordsRemover
words_remover = StopWordsRemover()
stop_words = words_remover.getStopWords()
sumasomaWords_RDD=cleaned_tokens_RDD.filter(lambda wrds: wrds not in stopwords)
```
### step 7:
We'll filter out everything that looks like an empty element to get rid of any empty elements.
```
# removing all the empty spaces from the data
suma_RemoveSpace_RDD = sumasomaWords_RDD.filter(lambda x: x != "")
```
## Process the data
### step 8:
We'll start by mapping our words into intermediate key-value pairs. e.g. (word,1). We move on to the next step, which involves converting the pairs into a word count.
```
#maps the words to key value pairs
IKVPairsRDD= suma_RemoveSpace_RDD.map(lambda word: (word,1))

# reduceByKey() to get (word,count) results
sumasoma_wordcount_RDD = IKVPairsRDD.reduceByKey(lambda acc, value: acc+value)
```
### step 9:
### step 10:
The collect() is used to retrieve all the elements from my dataset.
## Charting Results:
Pandas, MatPlotLib, and Seaborn has been used to visualize our performance.
# References:


