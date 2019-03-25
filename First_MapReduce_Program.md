# Run第一个MapReduce程序
> 参考：
> - Hadoop & Mapreduce Examples: Create your First Program https://www.guru99.com/create-your-first-hadoop-program.html#1

##### 在Master节点
```
# 切换到hadoop用户
su hadoop

# 创建项目
sudo mkdir MapReduceTutorial
sudo chmod -R 777 MapReduceTutorial
touch SalesMapper.java SalesCountryReducer.java SalesCountryDriver.java 
```

##### SalesMapper.java 
```
package SalesCountry;

import java.io.IOException;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapred.*;

public class SalesMapper extends MapReduceBase implements Mapper <LongWritable, Text, Text, IntWritable> {
	private final static IntWritable one = new IntWritable(1);

	public void map(LongWritable key, Text value, OutputCollector <Text, IntWritable> output, Reporter reporter) throws IOException {

		String valueString = value.toString();
		String[] SingleCountryData = valueString.split(",");
		output.collect(new Text(SingleCountryData[7]), one);
	}
}
```

##### SalesCountryReducer.java
```
package SalesCountry;

import java.io.IOException;
import java.util.*;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapred.*;

public class SalesCountryReducer extends MapReduceBase implements Reducer<Text, IntWritable, Text, IntWritable> {

	public void reduce(Text t_key, Iterator<IntWritable> values, OutputCollector<Text,IntWritable> output, Reporter reporter) throws IOException {
		Text key = t_key;
		int frequencyForCountry = 0;
		while (values.hasNext()) {
			// replace type of value with the actual type of our value
			IntWritable value = (IntWritable) values.next();
			frequencyForCountry += value.get();
			
		}
		output.collect(key, new IntWritable(frequencyForCountry));
	}
}

```

##### SalesCountryDriver.java 
```
package SalesCountry;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.*;
import org.apache.hadoop.mapred.*;

public class SalesCountryDriver {
    public static void main(String[] args) {
        JobClient my_client = new JobClient();
        // Create a configuration object for the job
        JobConf job_conf = new JobConf(SalesCountryDriver.class);

        // Set a name of the Job
        job_conf.setJobName("SalePerCountry");

        // Specify data type of output key and value
        job_conf.setOutputKeyClass(Text.class);
        job_conf.setOutputValueClass(IntWritable.class);

        // Specify names of Mapper and Reducer Class
        job_conf.setMapperClass(SalesCountry.SalesMapper.class);
        job_conf.setReducerClass(SalesCountry.SalesCountryReducer.class);

        // Specify formats of the data type of Input and output
        job_conf.setInputFormat(TextInputFormat.class);
        job_conf.setOutputFormat(TextOutputFormat.class);

        // Set input and output directories using command line arguments, 
        //arg[0] = name of input directory on HDFS, and arg[1] =  name of output directory to be created to store the output file.

        FileInputFormat.setInputPaths(job_conf, new Path(args[0]));
        FileOutputFormat.setOutputPath(job_conf, new Path(args[1]));

        my_client.setConf(job_conf);
        try {
            // Run the job 
            JobClient.runJob(job_conf);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

##### 设置CLASSPATH
```
export CLASSPATH="/opt/hadoop/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-client-core-2.7.6.jar:/opt/hadoop/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-client-common-2.7.6.jar:/opt/hadoop/hadoop/share/hadoop/common/hadoop-common-2.7.6.jar:/home/hadoop/MapReduceTutorial/SalesCountry/*"
```

##### 编译三个java文件生成jar包
```
# 编译
javac -d . SalesMapper.java SalesCountryReducer.java SalesCountryDriver.java
# 添加Manifest.txt文件
sudo gedit Manifest.txt中加如下行
Main-Class: SalesCountry.SalesCountryDriver（回车）

# 创建jar
jar cfm ProductSalePerCountry.jar Manifest.txt SalesCountry/*.class
```

##### 启动hadoop
```
$HADOOP_HOME/sbin/start-dfs.sh
$HADOOP_HOME/sbin/start-yarn.sh
```

##### 在HDFS上建input目录
```
$HADOOP_HOME/bin/hdfs dfs -copyFromLocal ~/inputMapReduce /

#注意：如果运行时产生warning: Unable to load native-hadoop library
#解决：在~/.bashrc文件中加入如下信息
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib:$HADOOP_COMMON_LIB_NATIVE_DIR"

$HADOOP_HOME/bin/hdfs dfs -ls /inputMapReduce
```

##### 运行任务
```
# 运行
$HADOOP_HOME/bin/hadoop jar ProductSalePerCountry.jar /inputMapReduce /mapreduce_output_sales
```

##### 查看结果
```
# 命令
$HADOOP_HOME/bin/hdfs dfs -cat /mapreduce_output_sales/part-00000
# GUI
ip:50070
```

##### 结果如下
```
Argentina	1
Australia	38
Austria	7
Bahrain	1
Belgium	8
Bermuda	1
Brazil	5
Bulgaria	1
CO	1
Canada	76
Cayman Isls	1
China	1
Costa Rica	1
Country	1
Czech Republic	3
Denmark	15
Dominican Republic	1
Finland	2
France	27
Germany	25
Greece	1
Guatemala	1
Hong Kong	1
Hungary	3
Iceland	1
India	2
Ireland	49
Israel	1
Italy	15
Japan	2
Jersey	1
Kuwait	1
Latvia	1
Luxembourg	1
Malaysia	1
Malta	2
Mauritius	1
Moldova	1
Monaco	2
Netherlands	22
New Zealand	6
Norway	16
Philippines	2
Poland	2
Romania	1
Russia	1
South Africa	5
South Korea	1
Spain	12
Sweden	13
Switzerland	36
Thailand	2
The Bahamas	2
Turkey	6
Ukraine	1
United Arab Emirates	6
United Kingdom	100
United States	462
```
