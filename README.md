# 2025_ITCS6190_Hands-on_L4

## o Project Overview

This project is a basic Hadoop MapReduce application that performs word count on a text file. It reads an input dataset, splits the text into individual words, ignores words shorter than three letters, and counts how many times each word appears. The program is written in Java and built into a JAR file using Maven. It runs on a Hadoop cluster that is set up inside Docker containers, making it easy to deploy and test in a consistent environment.

## o Approach and Implementation

### Mapper
It ignores any words that are shorter than three letters.
For each word it keeps, it sends out a pair that has the word itself and the number one.

### Reducer
For each word, the reducer takes all the numbers that come from the mapper. It adds these numbers together to get the total count of that word. Finally, it produces a pair that shows the word and its total count.

## o Execution Steps

### 1. Start Hadoop 

```bash
docker compose up -d
```

### 2. Build project

```bash
mvn clean package
```

### 3. Copy JAR to container

```bash
docker cp target/WordCountUsingHadoop-0.0.1-SNAPSHOT.jar resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/
```

### 4. Copy dataset to container

```bash
docker cp shared-folder/input/data/input.txt resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/
```

### 5. Connect to ResourceManager container

```bash
docker exec -it resourcemanager /bin/bash
cd /opt/hadoop-3.2.1/share/hadoop/mapreduce/
```

### 6. Upload dataset to HDFS

```bash
hadoop fs -mkdir -p /input/data
hadoop fs -put -f ./input.txt /input/data
```

### 7. Run MapReduce job

```bash
hadoop fs -rm -r -f /result
hadoop jar WordCountUsingHadoop-0.0.1-SNAPSHOT.jar com.example.controller.Controller /input/data/input.txt /result
```

### 8. View output

```bash
hadoop fs -ls /result
hadoop fs -cat /result/part-r-00000
```


## o Challenges Faced & Solutions

The first problem was a port conflict. The namenode container did not start because Portainer was already using port 9000. The solution was to change the port setting in `docker-compose.yml` so that the namenode used 9999 on the host.

The second problem was a build error caused by an old Java version. Maven tried to compile the code with Java 1.5, which is no longer supported. The solution was to update the `pom.xml` file to use Java 1.8 for compiling.



## o Input and Obtained Output

### Sample Input (`input.txt`)

```
UNC Charlotte is a university
Computer Science at UNC Charlotte
Cloud computing is popular
Students are at the library
UNC Charlotte has a program
The lab is at building
Cloud platforms are useful
Professors teach at the university
Technology is in the classroom
Machine learning is a subject
```

### HDFS Output Directory

```
root@f252fe4123e8:/opt/hadoop-3.2.1/share/hadoop/mapreduce# hadoop fs -ls /result
Found 2 items
-rw-r--r--   3 root supergroup          0 2025-09-09 19:50 /result/_SUCCESS
-rw-r--r--   3 root supergroup        252 2025-09-09 19:50 /result/part-r-00000
```

### Actual Output (`part-r-00000`)

```
the	3
UNC	3
Charlotte	3
university	2
Cloud	2
are	2
subject	1
platforms	1
useful	1
learning	1
Students	1
building	1
Machine	1
Science	1
teach	1
library	1
program	1
Computer	1
Professors	1
popular	1
lab	1
The	1
classroom	1
Technology	1
has	1
computing	1
```
