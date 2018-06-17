### NameNode
NameNode is also called the Master, which stores the metadata of HDFS.</br>
The following instructions will help you set up a namenode on Amazon EC2.</br>
#### Update the system & install Java 8 JDK
```
sudo apt-get update
sudo apt-get install default-jdk
```
#### Add JAVA_HOME environment variable
```
sudo touch /etc/profile.d/hadoop.sh
sudo echo 'export JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64"' | sudo tee /etc/profile.d/hadoop.sh
sudo chown ubuntu /etc/profile.d/hadoop.sh
```
#### Download Hadoop 3.0.3
```
wget http://www-us.apache.org/dist/hadoop/common/hadoop-3.0.3/hadoop-3.0.3.tar.gz -P ~/Downloads
```
#### Uncompress Hadoop 3.0.3
```
sudo tar xzvf ~/Downloads/hadoop-*.tar.gz -C /usr/local
```
rename "hadoop 3.0.3" directory located in /usr/local/ as "hadoop" and change the ownership of it to ubuntu
```
sudo mv /usr/local/hadoop-* /usr/local/hadoop
sudo chown -R ubuntu /usr/local/hadoop
```
#### Add HADOOP_HOME and HADOOP_CONF_DIR environment variables
```
sudo echo 'export HADOOP_HOME="/usr/local/hadoop"' | sudo tee --append /etc/profile.d/hadoop.sh
sudo echo 'PATH="$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin"' | sudo tee --append /etc/profile.d/hadoop.sh
sudo echo 'export HADOOP_CONF_DIR="/usr/local/hadoop/etc/hadoop"' | sudo tee --append /etc/profile.d/hadoop.sh
```
fill in private IP addresses and the name your private key; then COPY them to hadoop.sh
```
export namenodeIP="172.XXX.XXX.XXX"
export datanode1IP="172.XXX.XXX.XXX"
export IdentityFile="~/.ssh/YOUR_PRIVATE_KEY.pem"
```
reload the environment variables
```
source /etc/profile.d/hadoop.sh
```
#### Make a hard change to your hostname of NameNode
```
sudo hostname ${namenode}
sudo rm -rf /etc/hostname
echo -e "${namenode}" | sudo tee --append /etc/hostname
sudo chown root /etc/hostname
```
reboot your NameNode
```
sudo reboot
```
#### Set up config
```
sudo rm -rf ~/.ssh/config
sudo rm -rf ~/.ssh/known_hosts
echo "Host 0.0.0.0" | sudo tee --append ~/.ssh/config
echo "  HostName ${namenode}" | sudo tee --append ~/.ssh/config
echo "  User ubuntu" | sudo tee --append ~/.ssh/config
echo "  IdentityFile ${IdentityFile}" | sudo tee --append ~/.ssh/config
echo "Host namenode" | sudo tee --append ~/.ssh/config
echo "  HostName ${namenode}" | sudo tee --append ~/.ssh/config
echo "  User ubuntu" | sudo tee --append ~/.ssh/config
echo "  IdentityFile ${IdentityFile}" | sudo tee --append ~/.ssh/config
echo "Host datanode1" | sudo tee --append ~/.ssh/config
echo "  HostName ${datanode1}" | sudo tee --append ~/.ssh/config
echo "  User ubuntu" | sudo tee --append ~/.ssh/config
echo "  IdentityFile ${IdentityFile}" | sudo tee --append ~/.ssh/config
```
#### Edit core-site.xml
```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://0.0.0.0:9000</value>
    </property>
        <property>
        <name>hadoop.tmp.dir</name>
        <value>/tmp/hadoop-$(user.name)</value>
    </property>
</configuration>
```
#### yarn.xml
Referece: [Cloudera]: (https://www.cloudera.com/documentation/enterprise/5-14-x/topics/cdh_ig_yarn_cluster_deploy.html)
```
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```
#### mapred.xml
```
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.admin.user.env</name>
        <value>HADOOP_MAPRED_HOME=$HADOOP_COMMON_HOME</value>
    </property>
    <property>
        <name>yarn.app.mapreduce.am.env</name>
        <value>HADOOP_MAPRED_HOME=$HADOOP_COMMON_HOME</value>
    </property>
</configuration>
```
#### Create metadata directories for your Namenode and DataNode(s)
```
sudo mkdir -p $HADOOP_HOME/metadata/NameNode
sudo mkdir $HADOOP_HOME/metadata/DataNode
sudo chown -R ubuntu $HADOOP_HOME/metadata/
```
#### hdfs-site.xml
1 replication would be good enough since we only have 2 nodes here.</br>
The value of namenode.name.dir is the location of the metadata directory we created for the Master.  

```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
        <property>
                <name>dfs.namenode.name.dir</name>
                <value>usr/local/hadoop/metadata/NameNode</value>
        </property>
        <property>
                <name>dfs.datanode.data.dir</name>
                <value>usr/local/hadoop/metadata/DataNode</value>
        </property>
</configuration>
```
