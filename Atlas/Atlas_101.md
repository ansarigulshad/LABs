# ATLAS

##### Create users and groups on all nodes
```
# Add groups
groupadd engineering_grp
groupadd marketing_grp
groupadd support_grp


# Add Users
for i in e1 e2 e3 e4 e5 e6 ; do adduser $i -G engineering_grp; done
for i in m1 m2 m3 m4 ; do adduser $i -G marketing_grp; done
for i in s1 s2 s3 ; do adduser $i -G support_grp; done

# Verify
cat /etc/group | grep _grp
```

#####  Create hdfs user directory for new users
```
su - hdfs

for newuser in e1 e2 e3 e4 e5 e6 m1 m2 m3 m4 s1 s2 s3 
do
hdfs dfs -mkdir /user/$newuser ; hdfs dfs -chown $newuser /user/$newuser
done

hdfs dfs -ls /user
```

### STEP 1: Create Hive Tables
#### a) Download script & generate sample data file
```
yum clean all
yum install git -y

cd /var/tmp/
git clone https://github.com/ansarigulshad/script_datagenerator.git
cd script_datagenerator
chmod +x *.sh
```
```
sh data_generator.sh
```
```
head -50 employee_data.csv
```
#### b) Create Hive database & table

##### i. Login to hiveserver2 node and login beeline (change hive connection string as per your setup)
```
# HIVE_CONNECTION_STRING="jdbc:hive2://zk1.hortonworks.com:2181,zk2.hortonworks.com:2181,zk3.hortonworks.com:2181/;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=hiveserver2"

# beeline -n hive -u $HIVE_CONNECTION_STRING
```
##### ii. Create database
```
create database emp_sampledb;
use emp_sampledb;
```
##### iii. Create Table
```
create table employee(id int, name string, salary int, city string,mobile_no bigint) 
row format delimited
fields terminated by ','
stored as textfile;
```
```
!q
```
##### iv. Load data to `employee` table
```
sudo -su hive hdfs dfs -put \
/var/tmp/script_datagenerator/employee_data.csv \
/warehouse/tablespace/managed/hive/emp_sampledb.db/employee
```
For HDP-2.6.x Version
```
sudo -su hive hdfs dfs -put \
/var/tmp/script_datagenerator/employee_data.csv /apps/hive/warehouse/emp_sampledb.db/employee
```

#### d) Check employee table
```
# beeline -n hive -p hive -e "select * from emp_sampledb.employee limit 50" -u $HIVE_CONNECTION_STRING
```

### STEP 2: Create a Ranger Policy for Hive table
#### a) Go to Ranger UI & login with admin/admin credentials
#### b) Create hive policy for "employee" table and grant permission 

#### Access Hive table with `m1` user
```
# beeline -n m1 -p hive -e "select * from emp_sampledb.employee limit 50" -u $HIVE_CONNECTION_STRING
```
