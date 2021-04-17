# Big Data Final Project

## Text Data:
[Frankenstein; Or, The Modern Prometheus by Mary Wollstonecraft Shelley](https://www.gutenberg.org/cache/epub/42324/pg42324.txt)

## The Process:
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
3. 
