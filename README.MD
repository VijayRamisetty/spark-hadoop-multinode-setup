 Multi Node Hadoop & Spark Cluster 
====================================

 - Oracle VM box
 - Ubuntu 18.04 LTS .iso
 
 Target
 ======= 
-   (1) master (2) slaves  - Hadoop & Spark  Multi Node Cluster Setup inside Oracle VM box.
  
Install Oracle VM box
---------------------

- Load Ubuntu 18.04 LTS image
- Create user : vijay as admin

Setting Internet & Static IP address
------------------------------------
-   Vm box - > preferences ->  File -> Host network manager -> create new host-only network -> it will create vboxnet0 -> enable DHCP Server
Click on configure manually

-   Remember IP address format - 192.168.56.1 , ip4 network mask : 255.255.255.0
-   Ubuntu Vm -> settings- > network - > Adaptor-1 = NAT , -> Adapto-2 = Host only adaptor , vboxnet0 

Inside Ubuntu VM : 
---------------- 

    ifconfig -a 
    ip link show
    
    enp0s3 - using for internet
    enp0s8 - using for setting static ip
    cd /etc/netplan/


    vijay@ubuntu-hadoop-1:/etc/netplan$ cp 50-cloud-init.yaml 50-cloud-init.yaml.bkp
    sudo vim 50-cloud-init.yaml
    vijay@ubuntu-hadoop-master:/etc/netplan$ cat 50-cloud-init.yaml
    # This file is generated from information provided by
    # the datasource.  Changes to it will not persist across an instance.
    # To disable cloud-init's network configuration capabilities, write a file
    # /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
    # network: {config: disabled}
    network:
        ethernets:
            enp0s3:
                    addresses: []
                    dhcp4: true
            enp0s8:
                dhcp4: no
                addresses: [192.168.56.101/24]
                gateway4: 192.168.1.1
                nameservers:
                  addresses : [192.168.1.1 ,8.8.8.8]
        version: 2
    
    
      sudo netplan generate
      sudo netplan apply
      ifconfig -a
      ping 192.168.56.101
      hostname -i


Adding dedicated user
--------------------

    sudo addgroup hadoop
    sudo adduser -- ingroup hadoop hduser
    
    sudo visudo
    hduser ALL=(ALL:ALL) ALL
    
    su - hduser 



Change hostname
----------------
      sudo vim /etc/hostname  (or) 
      sudo hostnamectl set-hostname ubuntu-hadoop-master
      cat /etc/hostname 
      sudo vim /etc/hosts
      sudo /etc/cloud/cloud.cfg
      sudo vim /etc/cloud/cloud.cfg -> preserve_name to true

Installing Java 8 
------------------

    hduser@ubuntu-hadoop-master:/usr/local$ ls
    bin  games    jdk1.8.0_202                lib  sbin   src
    etc  include  jdk-8u202-linux-x64.tar.gz  man  share
    hduser@ubuntu-hadoop-master:/usr/local$ sudo mv jdk1.8.0_202/ java
    hduser@ubuntu-hadoop-master:/usr/local$ ls
    bin  games    java                        lib  sbin   src
    etc  include  jdk-8u202-linux-x64.tar.gz  man  share
    hduser@ubuntu-hadoop-master:/usr/local$ ls -ltr
    total 189532
    drwxr-xr-x 7 uucp      143      4096 Dec 15  2018 java
    drwxr-xr-x 2 root   root        4096 Feb 14  2019 src
    drwxr-xr-x 2 root   root        4096 Feb 14  2019 sbin
    lrwxrwxrwx 1 root   root           9 Feb 14  2019 man -> share/man
    drwxr-xr-x 2 root   root        4096 Feb 14  2019 include
    drwxr-xr-x 2 root   root        4096 Feb 14  2019 games
    drwxr-xr-x 2 root   root        4096 Feb 14  2019 etc
    drwxr-xr-x 2 root   root        4096 Feb 14  2019 bin
    drwxr-xr-x 3 root   root        4096 Feb 14  2019 lib
    drwxr-xr-x 4 root   root        4096 Feb 14  2019 share
    -rw-r--r-- 1 hduser hadoop 194042837 Apr 20 14:08 jdk-8u202-linux-x64.tar.gz
    hduser@ubuntu-hadoop-master:/usr/local$ cd ~
    hduser@ubuntu-hadoop-master:~$ vim .bashrc 
    hduser@ubuntu-hadoop-master:~$ tail .bashrc 
        . /usr/share/bash-completion/bash_completion
      elif [ -f /etc/bash_completion ]; then
        . /etc/bash_completion
      fi
    fi
    
    
    #JAVA_ENVIRONEMT
    export JAVA_HOME=/usr/local/java
    export PATH=$PATH:/usr/local/java/bin
    hduser@ubuntu-hadoop-master:~$ 
    hduser@ubuntu-hadoop-master:~$ ls /usr/bin/j
    join         jsonpatch    json_pp      
    jsondiff     jsonpointer  jsonschema   
    
    hduser@ubuntu-hadoop-master:~$ sudo update-alternatives --install "/usr/bin/java" "java" "/usr/local/java/bin/java" 1
    update-alternatives: using /usr/local/java/bin/java to provide /usr/bin/java (java) in auto mode
    hduser@ubuntu-hadoop-master:~$ sudo update-alternatives --install "/usr/bin/javac" "javac" "/usr/local/java/bin/javac" 1
    update-alternatives: using /usr/local/java/bin/javac to provide /usr/bin/javac (javac) in auto mode
    hduser@ubuntu-hadoop-master:~$ sudo update-alternatives --install "/usr/bin/javaws" "javaws" "/usr/local/java/bin/javaws" 1
    update-alternatives: using /usr/local/java/bin/javaws to provide /usr/bin/javaws (javaws) in auto mode
    
    hduser@ubuntu-hadoop-master:~$ sudo update-alternatives --set java /usr/local/java/bin/java
    hduser@ubuntu-hadoop-master:~$ sudo update-alternatives --set javac /usr/local/java/bin/javac
    hduser@ubuntu-hadoop-master:~$ sudo update-alternatives --set javaws /usr/local/java/bin/javaws
    hduser@ubuntu-hadoop-master:~$ 
    hduser@ubuntu-hadoop-master:~$ java -version
    java version "1.8.0_202"
    Java(TM) SE Runtime Environment (build 1.8.0_202-b08)
    Java HotSpot(TM) 64-Bit Server VM (build 25.202-b08, mixed mode)
    hduser@ubuntu-hadoop-master:~$



Password-less ssh
------------------

    hduser@ubuntu-hadoop-master:~$ ssh-keygen -t rsa
    Generating public/private rsa key pair.
    Enter file in which to save the key (/home/hduser/.ssh/id_rsa): 
    Enter passphrase (empty for no passphrase): 
    Enter same passphrase again: 
    Your identification has been saved in /home/hduser/.ssh/id_rsa.
    Your public key has been saved in /home/hduser/.ssh/id_rsa.pub.
    The key fingerprint is:
    SHA256:yu4fjzo8DAYcf/CmeagjqR3Xw5o6ykWoAf0luxJs2f0 hduser@ubuntu-hadoop-master
    The key's randomart image is:
    +---[RSA 2048]----+
    |                 |
    | .. .            |
    |...o.o.          |
    |..++.=+          |
    |..=o+*. S        |
    |.o..*=.o         |
    |..o+o=* E        |
    |+.=+ +=. +       |
    |++o+o.+=o .      |
    +----[SHA256]-----+
    hduser@ubuntu-hadoop-master:~$ ls .ssh/
    authorized_keys  id_dsa  id_dsa.pub  id_rsa  id_rsa.pub  known_hosts
    hduser@ubuntu-hadoop-master:~$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
    hduser@ubuntu-hadoop-master:~$ chmod og-wx ~/.ssh/authorized_keys
    hduser@ubuntu-hadoop-master:~$ ssh localhost
    Welcome to Ubuntu 18.04.2 LTS (GNU/Linux 4.15.0-45-generic x86_64)
    hduser@ubuntu-hadoop-master:~$ exit
    logout
    Connection to localhost closed.
    hduser@ubuntu-hadoop-master:~$


Creating installation directory
--------------------------------

    cd /usr/local
    sudo wget hadoop-2.7.7.tar.gz
    
    sudo tar xvzf hadoop-2.7.7.tar.gz
    sudo mv hadoop-2.7.7 hadoop
    
    sudo chown -R hduser:hadoop /usr/local/hadoop
    sudo chmod -R 777 /usr/local/hadoop


Disabling IPv6
---------------

    sudo nano /etc/sysctl.conf
    with…
    net.ipv6.conf.all.disable_ipv6=1
    net.ipv6.conf.default_ipv6=1
    net.ipv6.conf.lo.disable_ipv6=1
    
    cat /proc/sys/net/ipv6/conf/all/disable_ipv6


Hadoop path setting
-------------------

    sudo nano ~/.bashrc
    #HADOOP ENVIRONMENT 
    export HADOOP_PREFIX=/usr/local/hadoop export HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop export HADOOP_MAPRED_HOME=/usr/local/hadoop export HADOOP_COMMON_HOME=/usr/local/hadoop export HADOOP_HDFS_HOME=/usr/local/hadoop export YARN_HOME=/usr/local/hadoop export PATH=$PATH:/usr/local/hadoop/bin export PATH=$PATH:/usr/local/hadoop/sbin
    #HADOOP NATIVE PATH: 
    export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native export HADOOP_OPTS=“-Djava.library.path=$HADOOP_PREFIX/lib”


Setting up Hadoop Configurations
================================

1. hadoop-env.sh
    
        cd /usr/local/hadoop/etc/hadoop/
        
        
        export HADOOP_OPTS=-Djava.net.preferIPv4Stack=true
        export JAVA_HOME=/usr/local/java
        export HADOOP_HOME_WARN_SUPPRESS=“TRUE”
        export HADOOP_ROOT_LOGGER=“WARN”

2. yarn-site.xml

        <configuration>
        <property>
            <name>yarn.resourcemanager.hostname</name>
            <value>ubuntu-hadoop-master</value>
        </property>
        <property>
            <name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value>
        </property>
        <property>
            <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
            <value>org.apache.hadoop.mapred.ShuffleHandler</value>
        </property>
        
        </configuration>

3. hdfs-site.xml

        <configuration>
        <property>
            <name>dfs.replication</name>
            <value>2</value>
        </property>
        <property>
            <name>dfs.persmissions</name>
            <value>false</value>
        </property>	
        <property>
            <name>dfs.namenode.name.dir</name>
            <value>/usr/local/hadoop/yarn_data/hdfs/namenode</value>
        </property>
        <property>
            <name>dfs.datanode.data.dir</name>
            <value>/usr/local/hadoop/yarn_data/hdfs/datanode</value>
        </property>
        </configuration>
        </property>

4. core-site.xml

        <configuration>
        <property>
            <name>hadoop.tmp.dir</name>
            <value>/app/hadoop/tmp</value>
        </property>
        <property>
            <name>fs.default.name</name>
            <value>hdfs://ubuntu-hadoop-master:9000</value>
        </property>
        </configuration>

5. mapred-site.xml

        <configuration>
        <property>
            <name>mapred.framework.name</name>
            <value>yarn</value>
        </property>
        <property>
            <name>mapreduce.jobhistory.address</name>
            <value>ubuntu-hadoop-master:10020</value>
        </property>
        </configuration>



creating directories
--------------------
    sudo mkdir -p /app/hadoop/tmp
    sudo chown -R hduser:hadoop /app/hadoop/tmp
    sudo chmod -R 777 /app/hadoop/tmp
    
    sudo mkdir -p /usr/local/hadoop/yarn_data/hdfs/namenode sudo mkdir -p /usr/local/hadoop/yarn_data/hdfs/datanode sudo chmod -R 777 /usr/local/hadoop/yarn_data/hdfs/namenode sudo chmod -R 777 /usr/local/hadoop/yarn_data/hdfs/datanode sudo chown -R hduser:hadoop /usr/local/hadoop/yarn_data/hdfs/namenode sudo chown -R hduser:hadoop /usr/local/hadoop/yarn_data/hdfs/datanode

Clone  
------
- Take clone of master to slave1 ,slave2
- Ensure hostnames and ip addresses are correct
- Delete slaves file in slaves
- Delete namenode dir in slaves 

Name Node Format
----------------
 hdfs namenode -format
 
     hduser@ubuntu-hadoop-master:/usr/local/hadoop/yarn_data/hdfs/datanode$ hadoop namenode -format
     DEPRECATED: Use of this script to execute hdfs command is deprecated.
     Instead use the hdfs command for it.
     
     
     Formatting using clusterid: CID-f9b94014-eba4-48af-afc6-c85b44143546
     Re-format filesystem in Storage Directory /usr/local/hadoop/yarn_data/hdfs/namenode ? (Y or N) Y
     hduser@ubuntu-hadoop-master:/usr/local/hadoop/sbin$ ls
     distribute-exclude.sh  hdfs-config.sh           refresh-namenodes.sh  start-balancer.sh    start-yarn.cmd  stop-balancer.sh    stop-yarn.cmd
     hadoop-daemon.sh       httpfs.sh                slaves.sh             start-dfs.cmd        start-yarn.sh   stop-dfs.cmd        stop-yarn.sh
     hadoop-daemons.sh      kms.sh                   start-all.cmd         start-dfs.sh         stop-all.cmd    stop-dfs.sh         yarn-daemon.sh
     hdfs-config.cmd        mr-jobhistory-daemon.sh  start-all.sh          start-secure-dns.sh  stop-all.sh     stop-secure-dns.sh  yarn-daemons.sh


Start Demons
-------------
start-dfs.sh

    hduser@ubuntu-hadoop-master:/usr/local/hadoop/sbin$ ./start-dfs.sh 
    
    Starting namenodes on [ubuntu-hadoop-master]
    
    ubuntu-hadoop-master: starting namenode, logging to /usr/local/hadoop/logs/hadoop-hduser-namenode-ubuntu-hadoop-master.out
    ubuntu-hadoop-slave1: starting datanode, logging to /usr/local/hadoop/logs/hadoop-hduser-datanode-ubuntu-hadoop-slave1.out
    ubuntu-hadoop-slave2: starting datanode, logging to /usr/local/hadoop/logs/hadoop-hduser-datanode-ubuntu-hasoop-slave2.out
    0.0.0.0: starting secondarynamenode, logging to /usr/local/hadoop/logs/hadoop-hduser-secondarynamenode-ubuntu-hadoop-master.out
    
    hduser@ubuntu-hadoop-master:/usr/local/hadoop/sbin$ jps
    6498 SecondaryNameNode
    6613 Jps
    6255 NameNode

start-yarn.sh

    hduser@ubuntu-hadoop-master:/usr/local/hadoop/sbin$ ./start-yarn.sh 
    starting yarn daemons
    starting resourcemanager, logging to /usr/local/hadoop/logs/yarn-hduser-resourcemanager-ubuntu-hadoop-master.out
    ubuntu-hadoop-slave1: starting nodemanager, logging to /usr/local/hadoop/logs/yarn-hduser-nodemanager-ubuntu-hadoop-slave1.out
    ubuntu-hadoop-slave2: starting nodemanager, logging to /usr/local/hadoop/logs/yarn-hduser-nodemanager-ubuntu-hasoop-slave2.out

List running services
----------------------
    hduser@ubuntu-hadoop-master:/usr/local/hadoop/sbin$ jps
    6498 SecondaryNameNode
    6675 ResourceManager
    6926 Jps
    6255 NameNode
    
    hduser@ubuntu-hadoop-master:/usr/local/hadoop/sbin$ free -m
                  total        used        free      shared  buff/cache   available
    Mem:           1993         468        1190           0         334        1382
    Swap:          2116           0        2116

Run Example
------------
    yarn jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.7.jar pi 16 1000


Validating Setup
----------------
    hdfs dfsadmin -report
    yarn node -list
    hdfs dfs -mkdir -p /user/hadoop
URLs
-----
    http://master:50070/dfshealth.html#tab-overview
    http://ubuntu-hadoop-master:8088/cluster/nodes

Testing hadoop setup running wordcount example
-----------------------------------------------
    hdfs dfs -mkdir -p /user/hadoop/books
    mkdir example_workspace
    Cd example_Workspace
    
    wget -O alice.txt https://www.gutenberg.org/files/11/11-0.txt
    wget -O holmes.txt https://www.gutenberg.org/files/1661/1661-0.txt
    wget -O frankenstein.txt https://www.gutenberg.org/files/84/84-0.txt
    
    hdfs dfs -put alice.txt holmes.txt frankenstein.txt /user/hadoop/books
    
    hduser@ubuntu-hadoop-master:/usr/local/hadoop/example_workspace$ hdfs dfs -ls /user/hadoop/books
    Found 3 items
    -rw-r--r--   2 hduser supergroup     174481 2020-04-21 08:35 /user/hadoop/books/alice.txt
    -rw-r--r--   2 hduser supergroup     450783 2020-04-21 08:35 /user/hadoop/books/frankenstein.txt
    -rw-r--r--   2 hduser supergroup     607788 2020-04-21 08:35 /user/hadoop/books/holmes.txt
    hduser@ubuntu-hadoop-master:/usr/local/hadoop/example_workspace$ 
    
    
    yarn jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.7.jar wordcount "/user/hadoop/books/*" /user/hadoop/output

References:
-----------
-   https://www.linode.com/docs/databases/hadoop/how-to-install-and-set-up-hadoop-cluster/
-   https://www.linode.com/docs/databases/hadoop/install-configure-run-spark-on-top-of-hadoop-yarn-cluster/



Install Spark on existing hadoop yarn cluster ( on master node)
---------------------------------------------------------------

    cd /usr/local/hadoop/
    cd /usr/local/
    wget  http://apachemirror.wuchna.com/spark/spark-2.4.5/spark-2.4.5-bin-hadoop2.7.tgz 
    
    tar -xvf spark*.tgz
    mv spark-2.4.5-bin-hadoop2.7 spark
    sudo chown -R hduser:hadoop /usr/local/spark
    sudo chmod -R 777 /usr/local/spark


Setting Spark Env
--------------------
    sudo vim ~/.bashrc
    #SPARK ENVIRONMENT
    export SPARK_HOME=/usr/local/spark
    export PATH=$PATH:$SPARK_HOME/bin
    
    export LD_LIBRARY_PATH=$HADOOP_COMMON_LIB_NATIVE_DIR
    
    source ~/.bashrc
Setting Spark Properties in env
---------------------------------
    mv $SPARK_HOME/conf/spark-defaults.conf.template $SPARK_HOME/conf/spark-defaults.conf
Edit $SPARK_HOME/conf/spark-defaults.conf and set spark.master to yarn:

    vim spark-defaults.conf 
     
    # Example:
    # spark.master                     spark://master:7077
    spark.master yarn
    # spark.eventLog.enabled           true
    # spark.eventLog.dir               hdfs://namenode:8021/directory
    # spark.serializer                 org.apache.spark.serializer.KryoSerializer
    # spark.driver.memory              5g
    spark.driver.memory	512m
    spark.executor.memory	512m
    # spark.executor.extraJavaOptions  -XX:+PrintGCDetails -Dkey=value -Dnumbers="one two three"
    
    spark.eventLog.enabled	true
    spark.eventLog.dir	hdfs://ubuntu-hadoop-master:9000/spark-logs
    
    #History server
    spark.history.provider            org.apache.spark.deploy.history.FsHistoryProvider
    spark.history.fs.logDirectory     hdfs://ubuntu-hadoop-master:9000/spark-logs
    spark.history.fs.update.interval  10s
    spark.history.ui.port             18080

create
-------- 
    hdfs dfs -mkdir /spark-logs

start history server
-----------------------
    $SPARK_HOME/sbin/start-history-server.sh
Access Spark History Server
----------------------------
    http://ubuntu-hadoop-master:18080/

Test a sample spark job
--------------------------

    spark-submit --deploy-mode client \
                   --class org.apache.spark.examples.SparkPi \
                   $SPARK_HOME/examples/jars/spark-examples_2.11-2.4.5.jar 10


URLs:
----

-   Spark History Server   : http://ubuntu-hadoop-master:18080/
-   Hadoop Nodes / Tracker : http://ubuntu-hadoop-master:8088/cluster/nodes
-   Hadoop Hdfs 		   : http://master:50070/dfshealth.html#tab-overview

