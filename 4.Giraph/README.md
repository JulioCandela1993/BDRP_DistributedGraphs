# Big Data Research Project: Distributed Graph Processing in Giraph 

## Installing Giraph on top of Hadoop
Hadoop and Giraph are implemented in Java. They can be configured in Linux, Windows, or Mac OS. 
In this tutorial, we will be discussing the installation of Giraph and Hadoop for Ubuntu 20.04 LTS. If you are a user of Windows,
you can install Ubuntu in Virtual Box. 

Giraph requires Oracle JDK version 7 or higher. As Giraph jobs are executed by the same JVM that runs the Hadoop framework, 
Hadoop also needed to be deployed using JDK 7 or above. OpenJDK which is the open-source version of Oracle's JDK is also fine. 
This tutorial uses OpenJDK 8.

### Setting up the JAVA_HOME and PATH
After installing Java, we also need to make sure that the installation tree is available to the applications
through the environment variable `JAVA_HOME`. We can also set the location of binaries using the `PATH` variable. On Unix platforms,
 it is convenient to set up those variables in a global shell startup file `~/.bashrc`. We can run the following terminal commands or add the commands to the `~/.bashrc` file and `source` it using the command `source ~/.bashrc`.
 
```shell script
#see the path of java
~$ update-alternatives --list java
/usr/lib/jvm/java-1.8.0-openjdk-amd64/bin/java
~$ JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64
~$ export JAVA_HOME
~$ PATH=$JAVA_HOME/bin:$PATH
~$ export PATH
```
Check whether the `JAVA_HOME` is set by running the `echo $JAVA_HOME` command which should print the value of the variable.

### Downloading compatible Hadoop and Giraph

In this tutorial, we will be installing Giraph-1.1.0. Giraph binaries have a strong dependency on a specific version of Hadoop. 
We need to download the Giraph binary and extract it to know which version of Hadoop it is built on. The binaries are named as `giraph-x.x.x-for-hadoop-x.x.x`.
The Giraph-1.1.0 is built on Hadoop-1.2.1. We need to install this version of Hadoop to work with Giraph.  

We create a directory named `dist` in the `Home` directory to install Giraph and Hadoop. Then we download the Hadoop and Giraph in the directory and extract them.
```shell script
$ mkdir ~/dist
$ cd ~/dist
~/dist$ wget https://miroir.univ-lorraine.fr/apache/giraph/giraph-1.1.0/giraph-dist-1.1.0-bin.tar.gz
~/dist$ tar xzf giraph-dist-1.1.0-bin.tar.gz
~/dist$ wget https://archive.apache.org/dist/hadoop/common/hadoop-1.2.1/hadoop-1.2.1.tar.gz
~/dist$ tar xzf Hadoop-1.2.1.tar.gz
```
### Setting up Hadoop and Giraph environment
Now we have a binary installation of Hadoop. We need to expose them to command-line utilities. 
Additionally, we can add the variables to the `~/.bashrc` file and `source` it as we did for `JAVA_HOME`.
  
```shell script
~/dist$ HADOOP_HOME=~/dist/hadoop-1.2.1
~/dist$ export HADOOP_HOME
~/dist$ PATH=$HADOOP_HOME/bin:$PATH
~/dist$ export PATH
~/dist$ hadoop version
Hadoop 1.2.1
Subversion https://svn.apache.org/repos/asf/hadoop/common/branches/branch-1.2 -r 1503152
Compiled by mattf on Mon Jul 22 15:23:09 PDT 2013
From source with checksum 6923c86528809c4e7e6f493b6b413a9a
This command was run using /home/rony/dist/hadoop-1.2.1/hadoop-core-1.2.1.jar
```

This indicates that Hadoop is installed correctly and it can locate Java to run Hadoop commands.
Similarly, we need to configure the Giraph environment as follows. 

```shell script
~/dist$ GIRAPH_HOME=~/dist/giraph-1.1.0-for-hadoop-1.2.1
~/dist$ export GIRAPH_HOME
~/dist$ PATH=$GIRAPH_HOME/bin:$PATH
~/dist$ export PATH
~/dist$ export HADOOP_CONF_DIR=$GIRAPH_HOME/conf
~/dist$ giraph
Usage: giraph [-D<Hadoop property>] <jar containing vertex> <parameters to jar>
At a minimum one must provide a path to the jar containing the vertex to be executed.
``` 
### Configuring of Hadoop and Giraph
Giraph does not strictly require any settings in its XML configuration file as required configuration options can be passed to Giraph via its command-line utility. However, it is more convenient to put the common configurations in the giraph-site.xml file instead of typing them every time in the command prompt.

Although Giraph has its configuration file, it requires Hadoop to be configured correctly by Hadoop's configuration files. Hadoop spreads its configuration over different files, namely core-site.xml, hdfs-site.xml, and
mapred-site.xml with yarn-site.xml being an additional file for configuring YARN as part of Hadoop 2.x. Now we will be configuring the `Pseudo-Distributed` mode of Hadoop and Giraph.
Edit the Hadoop configuration file located in the `HADOOP_HOME/conf` as follows.

Edit the **core-site.xml** file as follows.
```xml
<configuration>
   <property>
      <name>hadoop.tmp.dir</name>
      <value>/home/UBUNTU_USER_NAME/dist/tmpdata</value>
   </property>
   <property>
      <name>fs.default.name</name>
      <value>hdfs://localhost/</value>
   </property>
</configuration>
```
Replace `UBUNTU_USER_NAME` with the name of your Ubuntu user. Create a directory in the `dist` directory named `tmpdata` 
which will store the temporary data of Hadoop. Change it's permission so that it is readable and writeable by anyone.
This might be a big security issue in a real-life distributed setting.

```shell script
$ mkdir ~/dist/tmpdata
$ chmod -R 777 ~/dist/tmpdata
```
Now edit the configuration of **hdfs-site.xml** file as follows. 
```xml
<configuration>
   <property>
      <name>dfs.replication</name>
      <value>1</value>
   </property>
</configuration>
```
Finally, we need to configure **mapred-site.xml** file as shown below. Rename `mapred-site.xml.template` to `mapred-site.xml` if necessary. Yarn was introduced in Hadoop-2.x and hence no configuration file related to Yarn for our version of Hadoop.
```xml
<configuration> 
   <property> 
      <name>mapred.job.tracker</name> 
      <value>localhost:8021</value> 
   </property> 
</configuration>
```
We need to provide the `JAVA_HOME` path in the **hadoop-env.sh** file. Append the following line in the file.
```shell script
export JAVA_HOME="/usr/lib/jvm/java-1.8.0-openjdk-amd64"
```

With this, we have finished configuring Hadoop and Giraph. Let do some test to ensure everything is working fine.
First, we will format the `namenode` as it is the first usage after installation. From next time, we do not format it. 
Then we start the `namenode` and the `datanode` and change the permission of the root directory of the HDFS.

```shell script
#format the namenode during the first usage only
$ hadoop namenode –format
$ hadoop-daemon.sh start namenode
$ hadoop-daemon.sh start datanode
$ hadoop fs –chmod 777 /
``` 

Let do some basic operation with HDFS such as copy and deleting a file. If all the following commands work correctly, we can be sure that the pseudo-distributed HDFS setup is done successfully.

```shell script
$ hadoop fs –put /etc/hosts /test.file
$ hadoop fs –cat /test.file
$ hadoop fs –rm /test.file
```
The final step of checking is to start MapReduce and try to run some jobs. We can start the MapReduce by initiating the following commands.
```shell script
$ hadoop-daemon.sh start jobtracker
$ hadoop-daemon.sh start tasktracker
$ cd $HADOOP_HOME
```
As the MapReduce started, we can run some example-job by running the following command. The expected output is also shown below.
```shell script
$ hadoop jar *examples*jar pi 10 1000
....
Job Finished in 35.513 seconds
Estimated value of Pi is 3.14080000000000000000
```
If you get the output as shown, Congratulations! You are now all set to run cool Giraph applications.

### Giraph Project Setup

We need to build the executable `Jar` file from the application code to run it in MapReduce. Therefore, we create a project in (Maven or Ant) and use the configuration file to pull the dependencies and compiling code. If we choose Maven, the Project Object Model (pom.xml) definition 
of the dependencies should be as follows. 

```xml
<dependencies>
        <dependency>
            <groupId>org.apache.giraph</groupId>
            <artifactId>giraph-core</artifactId>
            <version>1.1.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-core</artifactId>
            <version>1.2.1</version>
        </dependency>
</dependencies>
```
We create the executable `Jar` file from the project by running the following command in the project directory.

```shell script
$ mvn package
```
If there is no problem in the code and dependency, this command will generate the `Jar` file which we can run using MapReduce.

#### Reference
**Practical Graph Analytics with Apache Giraph** by Claudio Martella, Roman Shaposhnik, and Dionysios Logothetis.
