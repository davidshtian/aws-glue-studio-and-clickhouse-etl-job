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

You could also use Secrets Manager for saving these information, if so remember to add IAM policies for Secrets Manager:
![image](https://github.com/davidshtian/aws-glue-studio-and-clickhous-etl-job/assets/14228056/eeb7f2d4-4b98-443b-84d2-25aab4373ead)

> Notes: Please use the Username and Password for specific format, refer to https://tibco-supportdesk.nttcoms.com/hc/ja/articles/11570611724185-AWS-Glueから接続する方法.

> Notes: Using the additional key/value parameters for user and password is not working, due to extra "&" in the string, as shown as below:
![image](https://github.com/davidshtian/aws-glue-studio-and-clickhous-etl-job/assets/14228056/5a29b392-cfb5-4e36-b456-d64217f69912)

> Notes: Please provide user and password to connection string, otherwise you will see error "Glue ETL Marketplace: Either user/password or secretId should be provided for JDBC connector" in the job.

> Notes: If you're connecting to ClickHouse within VPC, please refer to https://docs.aws.amazon.com/glue/latest/dg/start-connecting.html and use the subnet with NAT GW.

## Glue ClickHouse Read Job
Add ClickHouse data source, apply mapping and S3 data destination:
<img width="1117" alt="image" src="https://github.com/davidshtian/aws-glue-studio-and-clickhous-etl-job/assets/14228056/52906e46-7314-4157-a43c-7e2c193045f8">

Check S3 data:
<img width="680" alt="image" src="https://github.com/davidshtian/aws-glue-studio-and-clickhous-etl-job/assets/14228056/07a1931b-ff22-44ea-9497-e54759a5940f">

> Notes: Please note the schema and schema mapping.

## Glue ClickHouse Write Job
Clean data first (as I used same output data above write back to table):
```
INSERT INTO tb1 VALUES (2, 'alice');

DELETE FROM tb1 WHERE uid = 1;

SELECT * FROM tb1

Query id: d10e1f66-c417-4419-bb98-d94b4c3cae50

┌─uid─┬─name──┐
│   2 │ alice │
└─────┴───────┘

1 row in set. Elapsed: 0.002 sec.
```

Add S3 data source, apply mapping and ClickHouse data destination:
<img width="1114" alt="image" src="https://github.com/davidshtian/aws-glue-studio-and-clickhous-etl-job/assets/14228056/1211e256-2083-463b-b9c4-9cfac93e9106">

Check ClickHouse data:
```
SELECT *
FROM tb1

Query id: d1a64288-6f55-4abf-86d3-5eb13cb099e0

┌─uid─┬─name──┐
│   2 │ alice │
└─────┴───────┘
┌─uid─┬─name──┐
│   1 │ david │
└─────┴───────┘
```

## Other Considerations
Tried other CSV data and see errors below:
<img width="697" alt="image" src="https://github.com/davidshtian/aws-glue-studio-and-clickhous-etl-job/assets/14228056/b7a9f76d-d43a-436a-a86f-55a46c32578f">

This might be related to schema infer, and workaround is to use schema infer within Glue and all the columns will be "String" and use apply mapping to turn to right data type.
![image](https://github.com/davidshtian/aws-glue-studio-and-clickhous-etl-job/assets/14228056/5bb1af49-20b5-441d-9454-2a4d455e2a82)

Decimal data type in CSV file cannot converted to Decimal correctly, but it will success converting to Double in Glue.
<img width="682" alt="image" src="https://github.com/davidshtian/aws-glue-studio-and-clickhous-etl-job/assets/14228056/d7235201-6f41-4685-ad99-36fd77eeacbb">
