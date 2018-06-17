### Hadoop 3.0.3 on Amazon EC2
The general idea is to help others who would like to learn how to host the latest version of Hadoop on Amazon EC2.

#### Prerequisites on your computer (Windows)
- Putty
- WinSCP

#### Prerequisites on Amazon EC2
- Instances: 2 (at least)
  - One for NameNode and the other for DataNode
- OS: Ubuntu (Linux)
- Security Group:
  1. Protocol: SSH source: anywhere and
  2. Protocol all traffic source: anywhere
- Key Pairs: public/private key (.pem)
