### Hadoop 3.0.3 on Amazon EC2
The general idea is to help others who would like to host the latest version of Hadoop on Amazon EC2.
#### Prerequisites on your computer (Windows)
- Putty
- WinSCP

#### Prerequisites on Amazon EC2
- Instances: 2 (at least)
    - One for NameNode and the other for DataNode
- OS: Ubuntu (Linux)
- Security Group:
    1. Protocol: SSH, source: anywhere
    2. Protocol: all traffic, source: anywhere
- Key Pairs: public/private key (.pem)

#### References
- [Hadoop Cluster Setup](http://hadoop.apache.org/docs/r3.0.3/hadoop-project-dist/hadoop-common/ClusterSetup.html#Hadoop_Startup)
