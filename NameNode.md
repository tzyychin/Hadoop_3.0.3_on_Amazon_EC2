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
rename hadoop3.0.3 located in /usr/local/ as hadoop and change the ownership to ubuntu (this is very important because it would save you from typing chown commands)
```
sudo mv /usr/local/hadoop-* /usr/local/hadoop
sudo chown -R ubuntu /usr/local/hadoop
```
#### Add HADOOP_ Environment variables
```
sudo echo 'export HADOOP_HOME="/usr/local/hadoop"' | sudo tee --append /etc/profile.d/hadoop.sh
sudo echo 'PATH="$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin"' | sudo tee --append /etc/profile.d/hadoop.sh
sudo echo 'export HADOOP_CONF_DIR="/usr/local/hadoop/etc/hadoop"' | sudo tee --append /etc/profile.d/hadoop.sh
```
COPY the following lines with their respective private IPs (you can find them on your instance dashboard) and your private key to hadoop.sh
```
export namenodeIP="172.XXX.XXX.XXX"
export datanode1IP="172.XXX.XXX.XXX"
export IdentityFile="~/.ssh/YOUR_PRIVATE_KEY.pem"
```
use the following command to load the environment variables
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
#### create metadata directories for your Namenode and DataNode(s)
```
sudo mkdir -p $HADOOP_HOME/metadata/NameNode
sudo mkdir $HADOOP_HOME/metadata/DataNode
sudo chown -R ubuntu $HADOOP_HOME/metadata/
```
