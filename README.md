
# AWS End to End Live Data Streaming of IOT Sensors and Analysis
This repository provides a comprehensive guide and setup for streaming realtime data from IoT sensors, logging it using industry-standard practices, and analyzing the data using Parquet format. The setup includes components like an Android/iOS app for sending real life and real-time data.

Applications : Efficiency of Data Storage as Parquet format is light weight.\
Reduced Cost of Data Storage.\
Valid Format for Hands-on analysis for data via Athena,QuickSight,etc.


## AWS Services Used
Lambda, API Gateway,S3 ,Glue Catalog, Athena, Kinesis Streams and Firehose
## Steps to follow
1. Sensor Logger Application
i.Install Sensor Logger App in your Android/IOS
Android-- https://play.google.com/store/apps/details?id=com.kelvin.sensorapp&hl=en_CA&gl=US \
IOS-- https://apps.apple.com/us/app/sensor-logger/id1531582925 \
ii. Follow the Steps in the photo given below\
Step 1. Enable for Ambient Sensor for this particular project(feel free to customize)\
Step 2. Go to Settings and Refer the image below enable http push , set the url the one which you will get while deploying the API 
![Architecture Diagram](https://via.placeholder.com/468x300?text=App+Screenshot+Here)

iii. This Application will send the data in JSON format to API Gateway
```bash
	{'light_illumination': 40.82624816894531, 'capture_time': 1712020418971037400}
```


2. Create a RestAPi using API Gateway
i.Create a REST API \
ii.Create a resource named data , enable cors and lambda integration. 
iii.After creating deploy the API and save the url in the app (refer 1.Sensor Logger Application)



3. Create a lambda function with Python as enviroment and uplod the code. 
i.Increase the default execution of lambda from 3sec to as per your needs (3-5Minutes). (Use Kafka in Ec2 instace for continous delivery)
ii.Create a IAM ROLE to allow permissions to lambda to have full access of Kinesis Streams to write
iii. Deploy the Lambda
```bash
import json
from time import sleep
from json import dumps
import random
import boto3

def lambda_handler(event, context):
    client = boto3.client('kinesis')
    payload_part=json.loads(event['body'])['payload']
    for i in payload_part:
        light_illumination=i['values']['lux']
        capture_time=i['time']
        data={"light_illumination":light_illumination,"capture_time":capture_time}
        print(data)
        client.put_record(
            StreamName="DataLogger",
            Data=json.dumps(data),
            PartitionKey="1"
        )
        
    return {
        'statusCode': 200,
        'body': json.dumps('Hello from Lambda!')
    }
```

4. Kinesis Data Streams
i.Create a Stream with Provisioned Mode and Shard Capacity as 1 (you can change as per your needs but make sure you follow proper sharding in lambda function)

5. S3 Buckets
i.Create 2 Buckets for Parquet Format and JSON Format (Single Bucket with Different Prefix can also be used)

6. Glue Catalog
Determining Structure of your data (JSON) is neccesary, if you wish to add more parameters in JSON by enabling multiple sensors. Make changes accordingly to Lambda and Catalog
i.Go to Athena\
ii.Make a default database.\
iii. Enter the Query (For this example I am only using ambient light sensor)
```bash
create external table default.tablename
(light_illumination float,
capture_time float)
stored as parquet
Location 's3://parquetbucketname/'
TBLPROPERTIES ("parquet.compression"="SNAPPY")
```

7.Go to Firehose\
i.Create a delivery stream\
ii.Select source as Kinesis\
iii.In source settings choose your kinesis stream which you have created\
iv.Select Destination as S3 Bucket\
v.Enable record format conversion and format as Parquet Apache\
vi.Select Glue Region where you created you glue and select database and table\
vii.Select destination as your s3 bucket and change the bucket size to 64MB as we are using parquet format is very light weight  , buffer interval to 60Sec\
viii.Enable Backup Setting and select your backup json bucket\
ix.Configure the backup bucket you created and set a prefix for your choice . (You can choose the same bucket as well)














## RoadBlock Faced and Solved
1.Make sure your lambda function is working correctly\
2.Make sure first you are able to recieve data in Streams\
3.Please check responses of API Gateway correctly.\
4.Select the sensor according to your needs and make sure you have correct json format and corresponding representation accross lambda and data catalog query schema
