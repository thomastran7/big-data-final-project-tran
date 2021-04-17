# Big Data Final Project

## Text Data:
[Frankenstein; Or, The Modern Prometheus by Mary Wollstonecraft Shelley](https://www.gutenberg.org/cache/epub/42324/pg42324.txt)

## The Process:

### Gathering Data
1. To start off, we will request or pull the data from the url of the text data using urllib.request library. Once the data is pulled, we will store the data in a temporary file. In this case we will pull the data from the Frankenstein text data and the temporary file will be called frank.txt.
```
import urllib.request
stringInURL = "https://www.gutenberg.org/cache/epub/42324/pg42324.txt"
urllib.request.urlretrieve(stringInURL, "/tmp/frank.txt")
```
2. Now that the data is stored, we will move the data in the temproary data into a different location using dbutils.fs.mv. The way dbutils.fs.mv works is by taking two arguements. First is what the file we want to move, the second is where we want the file to be moved. In this case we will move the file from temproary to data as shown below.
```
dbutils.fs.mv("file:/tmp/frank.txt", "dbfs:/data/frank.txt")
```
3. Finally, we will transfer the data file into Spark. This is done by taking the data file we have now and converting it into Spark's RDD (Resilient Distributed Datasets). As shown below using sc.textFile.
```
FranksRawRDD= sc.textFile("dbfs:/data/frank.txt")
```

### Cleaning the Data
The data that we have gathered at this point is filled with capitalized words, sentences, punctuations, and stopwords (words that don't add much meaning to a sentence Ex. the, have, etc.). </br>
</br>
4. To start the cleaning process, we will first break down the data with flatmapping. Here we will change any capitalized to a lower case, removing any spaces, and splitting up sentences into words.
```
FranksMessyTokensRDD = FranksRawRDD.flatMap(lambda line: line.lower().strip().split(" "))
```
5. Next is punctuation (.,!?, etc) To do this we will be using the re library. This library (re) uses a regular expression to look for anything that does not resemble a letter.
```
import re
FranksCleanTokensRDD = FranksMessyTokensRDD.map(lambda letter: re.sub(r'[^A-Za-z]', '', letter))
```
6. Once the punctuations are removed, the next step will be removing any stopwords. This is done with the use of StopWordsRemover library from pyspark.ml.feature. 
```
from pyspark.ml.feature import StopWordsRemover
remover = StopWordsRemover()
stopwords = remover.getStopWords()
FranksWordsRDD = FranksCleanTokensRDD.filter(lambda PointLessW: PointLessW not in stopwords)
```
7. *This next step isn't necessary, but just in case your data contains any empty elements (' '). To remove any empty elements, we simply just filter out anything that resembles an empty element.
```
FranksEmptyRemoveRDD = FranksWordsRDD.filter(lambda x: x != "")
```

### Processing the Data
After we finish up cleaning the data, we move on to proccessing our clean data. </br>
</br>
8. To start the process, we will map our words into intermediate key-value pairs. This will look like: (word,1), once we map it.
```
IKVPairsRDD = FranksEmptyRemoveRDD.map(lambda word: (word,1))
```
9. Next we will transform the pairs into a sort of word count. An example is say the word banana comes up once, this will in turn show up as (banana, 1). If banana shows up again, then it will show as (banana, 2).
```
FranksWordCountRDD = IKVPairsRDD.reduceByKey(lambda acc, value: acc + value)
```
10. Finally, we will take our list of words and order them by descending order by using sortByKey. Once they are ordered, we'll take the top 10 of our list, by using take. Finally we will then print our result to see the top 10 most used word in Frankenstein in descending order.
```
FranksResults = FranksWordCountRDD.map(lambda x: (x[1], x[0])).sortByKey(False).take(10)
print(FranksResults)
```

### Charting the Results
11. To visualize our results, we will use Pandas, MatPlotLib, and Seaborn.
```
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

source = 'Frankenstein; Or, The Modern Prometheus'
title = 'Top Words in ' + source
xlabel = 'Count'
ylabel = 'Words'

df = pd.DataFrame.from_records(FranksResults, columns =[xlabel, ylabel]) 
plt.figure(figsize=(10,3))
sns.barplot(xlabel, ylabel, data=df, palette="Greens_d").set_title(title)
```

### Outcome


