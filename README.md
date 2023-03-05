# Hadoop-Guide-for-Beginners
Apache Hadoop is a collection of open-source software utilities that facilitates using a network of many computers to solve problems involving massive amounts of data and computation. It provides a software framework for distributed storage and processing of big data using the MapReduce programming model. In this guide we will learn how to run our first project using Hadoop.

## How to install and run Hadoop using Docker
1. Make sure you have Docker and Docker-compose installed on your machine.
2. Clone this repository: `git clone https://github.com/big-data-europe/docker-hadoop`.
3. Build and run containers: `docker-compose up -d`. The `-d` flag is used to run the containers in the background.
4. To check if the containers are running: `docker ps`.
5. Create a directory for the input files and the code: `mkdir <example name>`.
6. Copy the input files to the HDFS: `docker cp <input file> namenode:/tmp/`.
7. Copy the code to the HDFS: `docker cp <code file>.java namenode:/tmp/`.
8. Open a shell in the name node container: `docker exec -it namenode /bin/bash`. You can now run HDFS commands.
9. Navigate to the directory where the code is located: `cd /tmp/`. For organization purposes, you can create a directory for the code and move the code to that directory: `mkdir <example name>` and `mv <code> <example name>`. You can also create an output directory fro the geneated classes: `mkdir classes`.
10. Export the HADOOP_CLASSPATH: `export HADOOP_CLASSPATH=$(hadoop classpath)`.
11. Create the required directories in the HDFS: 
    1. Create the root directory for this project: `hadoop fs -mkdir /<example name>`.
    2. Create the directory for the input files: `hadoop fs -mkdir /<example name>/Input`.
    3. Copy the input files to the HDFS: `hadoop fs -put /tmp/<input file> /<example name>/Input`.
12. Compile the code: `javac -classpath $HADOOP_CLASSPATH -d ./classes/ ./<code file>.java`. The `-d` flag is used to specify the directory where the classes are located and the `.` is used to specify the current directory.
13. Put the compiled code in a jar file: `jar -cvf <code file>.jar -C ./classes .`.
14. To run the code: `hadoop jar <code file>.jar <class name> /<example name>/Input/<input file> /<example name>/Output`. 
15. Check if the job was successful: `hadoop job -list all`. The job name is the second column of the output and the status is the third column.
16. To check the output: `hadoop fs -cat /<example name>/Output/*`. This will display the output in the terminal. To save the output to a file: `hadoop fs -cat /<example name>/Output/* > <output file>`.
17. To retrieve the output file from the HDFS: `docker cp namenode:/<example name>/Output/<output file> <output file>`.

>> You can open UI for HDFS  at `http://localhost:9870`.
>> You can find your HDFS files at `http://localhost:9870/explorer.html#/`. 

## Useful HDFS Commands:
- `hdfs dfs -ls`: list files and directories
- `hdfs dfs -copyFromLocal`: copy files from local to HDFS
- `hdfs dfs -put`: copy files from local to HDFS
- `hdfs dfs -mkdir`: create a directory
- `hdfs dfs -cp`: copy files from one HDFS path to another
- `hdfs dfs -mv`: move files from one HDFS path to another
- `hdfs dfs -rm -r`: remove a directory and its contents
- `hdfs dfs -chmod`: change permissions of a file or directory
- `hdfs dfs -touchz`: create an empty file
- `hdfs dfs -cat`: display the contents of a file
- `hdfs dfs -copyToLocal`: copy files from HDFS to local
- `hdfs dfs -get`: copy files from HDFS to local


## The wordcount example
- Create a file called `WordCount.java` with the following content:
```java
// Java imports
import java.io.IOException;
import java.util.StringTokenizer;

// Hadoop imports
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

// WordCount class
public class WordCount {

  // This is the mapper class
  public static class TokenizerMapper
       extends Mapper<Object, Text, Text, IntWritable>{

    private final static IntWritable one = new IntWritable(1); // To count the number of words
    private Text word = new Text(); // To store the word

    // This is the map function
    public void map(Object key, Text value, Context context
                    ) throws IOException, InterruptedException {
      StringTokenizer itr = new StringTokenizer(value.toString()); // Tokenize the input
      // For each word, emit the word and 1
      while (itr.hasMoreTokens()) {
        word.set(itr.nextToken());
        context.write(word, one);
      }
    }
  }

  // This is the reducer class
  public static class IntSumReducer
       extends Reducer<Text,IntWritable,Text,IntWritable> {
    private IntWritable result = new IntWritable(); // To store the sum of the words

    // This is the reduce function
    public void reduce(Text key, Iterable<IntWritable> values,
                       Context context
                       ) throws IOException, InterruptedException {
      int sum = 0; // To store the sum of the words
      // For each word, add the count
      for (IntWritable val : values) {
        sum += val.get();
      }
      result.set(sum); // Set the result
      context.write(key, result); // Emit the word and the sum
    }
  }

  // This is the main function
  public static void main(String[] args) throws Exception {
    Configuration conf = new Configuration(); // Create a new configuration
    Job job = Job.getInstance(conf, "word count"); // Create a new job
    job.setJarByClass(WordCount.class); // Set the jar by class
    job.setMapperClass(TokenizerMapper.class); // Set the mapper class
    job.setCombinerClass(IntSumReducer.class); // Set the combiner class
    job.setReducerClass(IntSumReducer.class); // Set the reducer class
    job.setOutputKeyClass(Text.class); // Set the output key class
    job.setOutputValueClass(IntWritable.class); // Set the output value class
    FileInputFormat.addInputPath(job, new Path(args[0])); // Set the input path
    FileOutputFormat.setOutputPath(job, new Path(args[1])); // Set the output path
    System.exit(job.waitForCompletion(true) ? 0 : 1); // Wait for the job to complete
  }
}
```
- Create a file called `input.txt` with the following content:
``` text
Mostafa
Wael
Mostafa
Kamal
Wael
Mohammed
Mohammed
Mostafa
Kamal
Wael
Mostafa
Mostafa
```

## Complete commands for the WordCount example:
1. Open a terminal and run the following commands:
    ```bash
    git clone https://github.com/big-data-europe/docker-hadoop
    docker-compose up -d
    ``` 
2. Open another terminal in the code directory and run the following commands:
    ``` bash
    docker cp ./WordCount.java namenode:/tmp/
    docker cp ./input.txt namenode:/tmp/
    docker exec -it namenode /bin/bash
    cd /tmp/
    mkdir wordcount
    mv WordCount.java ./wordcount/
    mv input.txt ./wordcount/
    cd wordcount/
    mkdir classes
    ls
    export HADOOP_CLASSPATH=$(hadoop classpath)
    hadoop fs -mkdir /wordcount
    hadoop fs -mkdir /wordcount/Input
    hadoop fs -put /tmp/wordcount/input.txt /wordcount/Input
    javac -classpath $HADOOP_CLASSPATH -d ./classes/ ./WordCount.java 
    jar -cvf WordCount.jar -C ./classes .
    hadoop jar WordCount.jar WordCount  /wordcount/Input/input.txt /wordcount/Output
    hadoop fs -cat /wordcount/Output/*
    hadoop fs -cat /wordcount/Output/* > output.txt
    cat output.txt 
    ```
3. Return to first terminal and run the following commands:
    ```bash
    docker cp namenode:/tmp/wordcount/output.csv output.csv
    docker-compose down
    ```
