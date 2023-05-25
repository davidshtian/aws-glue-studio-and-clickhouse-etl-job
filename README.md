# AWS Glue Studio + ClickHouse Integration
A quick guide for integrating Glue ETL job and ClickHouse on Glue Studio via custom JDBC driver and connection.

## ClickHouse Installation
Refer to https://clickhouse.com/docs/en/install.
```
curl https://clickhouse.com/ | sh
sudo ./clickhouse install
```

Start the server:
```
sudo clickhouse start
```

Connect to server:
```
$ clickhouse-client --user default --password 
ClickHouse client version 23.5.1.1908 (official build).
Connecting to localhost:9000 as user default.
Password for user (default): 
Connecting to localhost:9000 as user default.
Connected to ClickHouse server version 23.5.1 revision 54462.
```

Create database and table:
```
CREATE DATABASE glue;

USE glue;

CREATE TABLE tb1
(
    uid UInt32,
    name String
)
ENGINE = MergeTree
PRIMARY KEY (uid);

INSERT INTO tb1 VALUES (1, 'david');
```

## Download the JDBC driver
Refer to https://github.com/ClickHouse/clickhouse-java, download the driver and upload to S3:
```
wget https://repo1.maven.org/maven2/com/clickhouse/clickhouse-jdbc/0.4.5/clickhouse-jdbc-0.4.5.jar

aws s3 cp clickhouse-jdbc-0.4.5.jar s3://<your s3 bucket>/glue/clickhouse-jdbc-0.4.5.jar
```

## Add Glue connector/connection
Refer to https://docs.aws.amazon.com/glue/latest/ug/connectors-chapter.ht, create connector first:
![image](https://github.com/davidshtian/aws-glue-studio-and-clickhous-etl-job/assets/14228056/f84ec72a-ec3e-47e5-9602-b03bb5a22b90)

Create connection based on the connector:
![image](https://github.com/davidshtian/aws-glue-studio-and-clickhous-etl-job/assets/14228056/a993ec78-0b47-4688-bccf-1eef10650633)

You could also use Secret Store for saving these information:
![image](https://github.com/davidshtian/aws-glue-studio-and-clickhous-etl-job/assets/14228056/eeb7f2d4-4b98-443b-84d2-25aab4373ead)

> Notes: Please use the Username and Password for specific format, refer to https://tibco-supportdesk.nttcoms.com/hc/ja/articles/11570611724185-AWS-Glueから接続する方法.

> Notes: Using the additional parameters for user and password is not working, due to extra "&" in the string, as shown as below:
![image](https://github.com/davidshtian/aws-glue-studio-and-clickhous-etl-job/assets/14228056/5a29b392-cfb5-4e36-b456-d64217f69912)













