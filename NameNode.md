### NameNode
NameNode is also called the Master, which stores the metadata of HDFS.</br>
The following instructions will help you set up a namenode on Amazon EC2.</br>
#### Update The System & Install JAVA
```
sudo apt-get update
sudo apt-get install default-jdk
```
#### Add Java_HOME Environment Variable
```
sudo touch /etc/profile.d/hadoop.sh
sudo echo 'export JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64"' | sudo tee /etc/profile.d/hadoop.sh
source /etc/profile.d/hadoop.sh
```
#### Download Hadoop 3.0.3
```
wget http://www-us.apache.org/dist/hadoop/common/hadoop-3.0.3/hadoop-3.0.3.tar.gz -P ~/Downloads
```
