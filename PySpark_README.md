#Apache Spark and MySQL


## Apache Spark Setup
### Verify Java
You will need to be sure you have java running on your system. If you have a fresh install of Ubuntu this may not be the case. To check, run the following command.
```
java -version
```

If you do not have it, get the default Java package.
```
sudo apt-get install default-jre
```

### Download Spark
To start out you will need to download Apache Spark.
The download I am for this README can be found [here](https://www.apache.org/dyn/closer.lua/spark/spark-2.3.0/spark-2.3.0-bin-hadoop2.7.tgz).

We will also need to get MySQL, but we will do that a bit later on.

## Extracting and Installing Spark
Now that you have Spark downloaded, you can extract it from either your file explorer or you with this command. 
```
tar xvf spark-1.3.1-bin-hadoop2.6.tgz 
```

You will next need to move the files to its own directory in **/usr/local/spark**.
```
sudo mv spark-2.3.0-bin-hadoop2.7/ /usr/local/spark
```

Now we need to prepare the environment.
```
export PATH = $PATH:/usr/local/spark/bin
source ~/.bashrc
```

### Verify the installation.

Start by opening the Spark shell
```
spark-shell
```
If successful, you should end up with this output once everything has booted up.
```
Welcome to 
      ____              __ 
     / __/__  ___ _____/ /__ 
    _\ \/ _ \/ _ `/ __/  '_/ 
   /___/ .__/\_,_/_/ /_/\_\   version 1.4.0 
      /_/  
		
Using Scala version 2.10.4 (Java HotSpot(TM) 64-Bit Server VM, Java 1.7.0_71) 
Type in expressions to have them evaluated. 
Spark context available as sc  
scala>
```

## MySQL setup
The next step will be getting our SQL server running. To start we will use the package manager to get MySQL Server.
```
sudo apt-get update
sudo apt-get install mysql-server
```
Run through the secure installation setup. I personally kept security to a minimum for ease of use.
```
mysql_secure_installation
```

Check to see if MySQL is running
```
systemctl status mysql.service
```
You should get the following output if the installation worked properly.
```
● mysql.service - MySQL Community Server
   Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset: en
   Active: active (running) since Wed 2016-11-23 21:21:25 UTC; 30min ago
 Main PID: 3754 (mysqld)
    Tasks: 28
   Memory: 142.3M
      CPU: 1.994s
   CGroup: /system.slice/mysql.service
           └─3754 /usr/sbin/mysqld
```
You can exit this display with Control+C

If you do not get this output, you can start the server manually with the following command.
```
sudo systemctl start mysql
```

### Setting up a demo database
We are now going to create a sample database to use with Spark.
Launch MySQL
```
mysql -u root -p
```


```
create database demo;
use demo;
create table demotable(id int,name varchar(20),age int);
insert into demotable values(1,"abhay",25),(2,"banti",25);
Select * from demotable;
```
**Note:** Do not exit out of MySQL for the next step. The SQL server must still be running while we use PySpark.

## Using Spark with Python (PySpark)
Now that we have Spark and our demo MySQL database running, let's connect the two.
You will first need to download the MySQL jar which contains all of the classes need to connect PySpark to a MySQL database.
You can get that file [here](http://www.java2s.com/Code/JarDownload/mysql/mysql.jar.zip) 

Extract the file in your desired location and launch PySpark with the following command in a new terminal.
```
pyspark --jars /path/to/mysql.jar
```

We will use the sqlContext command to load the table we made in our demo database as a frame.
**Note:** You will need to fill in the password value with the MySQL password you made in setup. 
```
dataframe_mysql = sqlContext.read.format("jdbc").options( url="jdbc:mysql://:3306/demo",driver = "com.mysql.jdbc.Driver",dbtable = "demotable",user="root", password="XXXXX").load()
```

Now display the new dataframe.
```
dataframe_mysql.show()
```

### Importing a database.
Download and extract the practice world database [here](http://downloads.mysql.com/docs/world.sql.zip).

Now, in your SQL terminal, create the new world database.
```
create database world;
source ~/Downloads/world.sql;
use world;
```

Verify that the tuples were set up correctly. You should see the listing of all of the cities.
```
select * from city;
```

Load the Country table using sqlContext.
```
Country = sqlContext.read.format("jdbc").options( url="jdbc:mysql://localhost:3306/world",driver = "com.mysql.jdbc.Driver",dbtable = "country",user="root", password="XXXXX").load()

Country.persist()
```

Load the CountryLanguage table using sqlContext.
```
CountryLanguage = sqlContext.read.format("jdbc").options( url="jdbc:mysql://localhost:3306/world",driver = "com.mysql.jdbc.Driver",dbtable = "countrylanguage",user="root", password="atMEGA328p").load().persist()
```

Check the headers for the dataframes.
```
Country.columns

CountryLanguage.columns
```

You should get these two outputs. 
```
>>> Country.columns
['Code', 'Name', 'Continent', 'Region', 'SurfaceArea', 'IndepYear', 'Population', 'LifeExpectancy', 'GNP', 'GNPOld', 'LocalName', 'GovernmentForm', 'HeadOfState', 'Capital', 'Code2']

>>> CountryLanguage.columns
['CountryCode', 'Language', 'IsOfficial', 'Percentage']
```
