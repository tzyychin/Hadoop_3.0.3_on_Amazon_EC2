### DataNode
DameNode is also called the Slave, which stores the actual data in HDFS
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
Rename "hadoop 3.0.3" directory located in /usr/local/ as "hadoop" and change the ownership of it to ubuntu
```
sudo mv /usr/local/hadoop-* /usr/local/hadoop
sudo chown -R ubuntu /usr/local/hadoop
```
#### Add HADOOP_HOME and HADOOP_CONF_DIR environment variables to hadoop.sh
```
sudo echo 'export HADOOP_HOME="/usr/local/hadoop"' | sudo tee --append /etc/profile.d/hadoop.sh
sudo echo 'PATH="$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin"' | sudo tee --append /etc/profile.d/hadoop.sh
sudo echo 'export HADOOP_CONF_DIR="/usr/local/hadoop/etc/hadoop"' | sudo tee --append /etc/profile.d/hadoop.sh
```
#### Add namenode and datanode(s) environement variables to hadoop.sh
```
export namenode="ec2-XXX-XXX-XXX-XXX.us-east-2.compute.amazonaws.com"
export datanode1="ec2-XXX-XXX-XXX-XXX.us-east-2.compute.amazonaws.com"
export namenodeIP="172.XXX.XXX.XXX"
export datanode1IP="172.XXX.XXX.XXX"
export IdentityFile="~/.ssh/YOUR_PRIVATE_KEY.pem"
sudo echo "export namenode=\"${namenode}\"" | sudo tee --append /etc/profile.d/hadoop.sh
sudo echo "export datanode1=\"${datanode1}\"" | sudo tee --append /etc/profile.d/hadoop.sh
sudo echo 'export namenodeIP=${namenodeIP}' | sudo tee --append /etc/profile.d/hadoop.sh
sudo echo 'export datanode1IP=${datanode1IP}' | sudo tee --append /etc/profile.d/hadoop.sh
sudo echo 'export IdentityFile=${IdentityFile}' | sudo tee --append /etc/profile.d/hadoop.sh

```
Reload the environment variables
```
source /etc/profile.d/hadoop.sh
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
sudo chown ubuntu ~/.ssh/config
```
#### Edit hosts file
```
sudo rm -rf /etc/hosts
echo "127.0.0.1 localhost" | sudo tee --append /etc/hosts
echo "${namenodeIP} namenode" | sudo tee --append /etc/hosts
echo "${datanode1IP} datanode1" | sudo tee --append /etc/hosts
```
You may skip the following lines for IPv6 hosts.
```
echo "# The following lines are desirable for IPv6 capable hosts" | sudo tee --append /etc/hosts
echo "::1 ip6-localhost ip6-loopback" | sudo tee --append /etc/hosts
echo "fe00::0 ip6-localnet" | sudo tee --append /etc/hosts
echo "ff00::0 ip6-mcastprefix" | sudo tee --append /etc/hosts
echo "ff02::2 ip6-allrouters" | sudo tee --append /etc/hosts
echo "ff02::3 ip6-allhosts" | sudo tee --append /etc/hosts
sudo chown root /etc/hosts
```
#### Edit core-site.xml (slightly different from that of namenode)
```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://namenode:9000</value>
    </property>
        <property>
        <name>hadoop.tmp.dir</name>
        <value>/tmp/hadoop-$(user.name)</value>
    </property>
</configuration>
```
#### Edit yarn.xml
```
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>namenode</value>
    </property>
</configuration>
```
#### Edit hdfs-site.xml
```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
        <property>
                <name>dfs.datanode.data.dir</name>
                <value>usr/local/apache/hadoop/metadata/DataNode</value>
        </property>
</configuration>
```
#### Edit mapred.xml
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
#### Generate the key fingerprint and add it to authorized_keys
```
sudo rm -rf ~/.ssh/id_rsa*
sudo rm -rf ~/.ssh/known_hosts
ssh-keygen -f ~/.ssh/id_rsa -t rsa -P ""
sudo chmod 0600 ~/.ssh/id_rsa.pub
sudo cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```
#### Add 0.0.0.0, namenode and datanode(s) to known_hosts
```
hosts=0.0.0.0,namenode,datanode1
ssh-keyscan -H ${hosts} >> ~/.ssh/known_hosts
```

#### Start hadoop-daemons, yarn-daemons if not running
```
sudo $HADOOP_HOME/sbin/hadoop-daemons.sh --config $HADOOP_CONF_DIR --script hdfs start datanode
sudo $HADOOP_HOME/sbin/yarn-daemon.sh --config $HADOOP_CONF_DIR start resourcemanager
sudo $HADOOP__HOME/sbin/yarn-daemons.sh --config $HADOOP_CONF_DIR start nodemanager
```
