# 📊 Real-Time Stock Market Data Streaming using Kafka – Step-by-Step 

---

## 🧠 What are we building?

We’re making a cool system that:
- Pretends to get live stock prices (like Apple stock)
- Sends that data into a pipe called **Kafka**
- Another part reads that data from the pipe and prints it

---

## 🧱 Step 1: Set up Your Playground (EC2 + Kafka)

### 🖥️ 1.1 Launch EC2 (Virtual Machine)
- Go to [AWS EC2 Console](https://console.aws.amazon.com/ec2/)
      - name : `kafka`
      - Amazon Machine Image (AMI) : `Amazon Linux 2023 kernel-6.1 AMI`
      - Instance type : `t2.micro`
      - Attach **IAM role** and **EC2 Client SG**
      - lanuch instance
2. connect instance and Configure Kafka on EC2

### 🛠️ 1.2 Install Kafka
```bash
sudo yum update -y
wget https://downloads.apache.org/kafka/3.6.0/kafka_2.13-3.6.0.tgz
tar -xzf kafka_2.13-3.6.0.tgz
cd kafka_2.13-3.6.0
```


## 5. 🛠 Configure Kafka on EC2

### Install Java & Kafka
```bash
sudo -i
sudo yum -y install java-11
# /3.6.0/ you need to replace with your kafka version of yout MSK
wget https://archive.apache.org/dist/kafka/3.6.0/kafka_2.13-3.6.0.tgz
tar -xzf kafka_2.13-3.6.0.tgz
cd kafka_2.13-3.6.0
```

### Add AWS MSK IAM Auth JAR
```bash
cd libs
wget https://github.com/aws/aws-msk-iam-auth/releases/download/v1.1.1/aws-msk-iam-auth-1.1.1-all.jar
cd ..
```

---

## 6. ⚙️ Create `client.properties`

Create a file:  
```bash
cd bin
vim client.properties
# (or)

nano /root/kafka_2.13-3.6.0/bin/client.properties

```
- paste this

```properties
security.protocol=SASL_SSL
sasl.mechanism=AWS_MSK_IAM
sasl.jaas.config=software.amazon.msk.auth.iam.IAMLoginModule required;
sasl.client.callback.handler.class=software.amazon.msk.auth.iam.IAMClientCallbackHandler
```
- Then press:
    - Ctrl + O to save
    - Enter to confirm
    - Ctrl + X to exit

```bash
ls
cd ..
```
---

## 7. 🧪 Create Kafka Topic

**NOTE :** before we are create this topic we need to make sure that msk cluster is up and running so because that is where we will be creating our kafka topic
#### step: 1
  - go to msk
       - open your clusters 
         - go to `View client information`
         - Bootstrap servers
         - Private endpoint (single-VPC) 
         - ***copy this (eg: b-1.mskcluster.sv5gwy.c3.kafka.ap-south-1.amazonaws.com:9098 )***

### step:2
 - Amazon MSK -> Clusters -> MSK-Cluster
 - Properties
 - Networking settings 
 - Security groups applied
 - juzt check the inbound rule
      
 ### step:3    
```bash
bin/kafka-topics.sh --create --bootstrap-server <bootstrapServerString> --command-config ./client.properties --replication-factor 3 --partitions 1 --topic MSKTutorialTopic

# replace <bootstrapServerString> to Private endpoint msk
bin/kafka-topics.sh \
--create \
--bootstrap-server b-1.mskcluster.sv5gwy.c3.kafka.ap-south-1.amazonaws.com:9098 \
--command-config /root/kafka_2.13-3.6.0/bin/client.properties \
--replication-factor 3 \
--partitions 1 \
--topic MSKTutorialTopics

```

### step:4 View All Kafka Topics in EC2 (Connected to MSK)
1. Navigate to your Kafka directory:
2. Run the list topics command:
```bash
bin/kafka-topics.sh \
--list \
--bootstrap-server b-1.mskcluster.sv5gwy.c3.kafka.ap-south-1.amazonaws.com:9098 \
--command-config ./bin/client.properties
```
***NOTE:Replace the --bootstrap-server value and the client.properties path with your actual MSK broker and config file***

3.📌 Example Output:
```bash
MSKTutorialTopic
MSKTutorialTopics
logs-topic
```
4. Count Topics
- To count how many topics exist, use:
```bash
bin/kafka-topics.sh \
--list \
--bootstrap-server b-1.mskcluster.sv5gwy.c3.kafka.ap-south-1.amazonaws.com:9098 \
--command-config ./bin/client.properties | wc -l

```

## 8:  📤 start Produce Messages
```bash
bin/kafka-console-producer.sh --broker-list <bootstrapServerString> --producer.config /root/kafka_2.13-3.6.0/bin/client.properties --topic MSKTutorialTopic

# replace <bootstrapServerString>

bin/kafka-console-producer.sh \
  --broker-list b-1.mskcluster.sv5gwy.c3.kafka.ap-south-1.amazonaws.com:9098 \
  --producer.config /root/kafka_2.13-3.6.0/bin/client.properties \
  --topic MSKTutorialTopics


```

Type messages and press Enter.
  - hi
  - hello 
  - hello
  - this is kafka
  - msk is working!



---
### step:9
  - connect instance in one tap and runthis commands

## 9. 📥 Consume Messages
```bash
bin/kafka-console-consumer.sh --bootstrap-server <bootstrapServerString> --consumer.config /root/kafka_2.13-3.6.0/bin/client.properties --topic MSKTutorialTopic --from-beginning


# replace <bootstrapServerString>

bin/kafka-console-consumer.sh \
--bootstrap-server b-1.mskcluster.sv5gwy.c3.kafka.ap-south-1.amazonaws.com:9098 \
--consumer.config /root/kafka_2.13-3.6.0/bin/client.properties \
--topic MSKTutorialTopics \
--from-beginning


```
1. Then the consumer will output:
  - hi
  - hello 
  - hello
  - this is kafka
  - msk is working!

2. 🛑 To Stop the Consumer
Just press: 
   ```bash
   Ctrl + C
   ```
---

---
## 📥 Step-by-Step: Download ca.pem File on EC2

### 🖥️ Step 1: Go to Kafka bin directory
```bash
cd /root/kafka_2.13-3.6.0/bin/
```
```bash
curl -o ca.pem https://www.amazontrust.com/repository/AmazonRootCA1.pem
```
- ✅ This will download the certificate and save it as ca.pem in your current directory.


```bash
ls -l ca.pem
```
- Expected output:
```bash
-rw-r--r-- 1 root root 1114 Jul 8 12:34 ca.pem
```

```bash
chmod 400 ca.pem
head ca.pem
```
- Now you're done! ✅ The file is ready to be used in your Kafka config:
```bash
ssl_cafile="/root/kafka_2.13-3.6.0/bin/ca.pem"
```

---

## 🧑‍🍳 Step 4: Create the Data Sender (Producer)
### 4.0 📦 Install Required Python Packages on EC2:
- ✅ Final Working Kafka Producer Script (producer.py)

```bash
sudo apt update
sudo yum install python3 -y
sudo yum install python3-pip -y
pip3 --version
#Install Required Python Libraries
pip3 install kafka-python pandas s3fs
pip3 install kafka-python pandas
pip3 install boto3
pip3 install confluent-kafka s3fs pandas


```
```bash
vim producer.py
```
### 4.1 Create a file called `producer.py`

```python
import pandas as pd
from kafka import KafkaProducer
from time import sleep
from json import dumps
import json
import s3fs

# ✅ Kafka MSK broker (SSL)
KAFKA_BROKERS = [
    'b-2.mycluster.lqdehq.c3.kafka.ap-south-1.amazonaws.com:9098',
    'b-1.mycluster.lqdehq.c3.kafka.ap-south-1.amazonaws.com:9098',
    'b-3.mycluster.lqdehq.c3.kafka.ap-south-1.amazonaws.com:9098'
]

# ✅ Create Kafka producer with SSL
producer = KafkaProducer(
    bootstrap_servers=KAFKA_BROKERS,
    security_protocol="SASL_SSL",
    value_serializer=lambda x: dumps(x).encode('utf-8')
)

# ✅ Send test message
producer.send('MSKTutorialTopics', value={'message': 'test from Python producer'})
producer.flush()
print("✅ Test message sent to topic MSKTutorialTopics.")

# ✅ Load CSV from S3
s3_path = "s3://yashdatabase/sourcedata(Producer)/indexProcessed.csv"
print(f"📂 Loading CSV from: {s3_path}")
s3 = s3fs.S3FileSystem()

with s3.open(s3_path, 'rb') as f:
    df = pd.read_csv(f)

print(f"✅ DataFrame loaded with {len(df)} rows. Streaming data to Kafka...")

# ✅ Stream data to Kafka topic
while True:
    dict_stock = df.sample(1).to_dict(orient="records")[0]
    producer.send('MSKTutorialtopics', value=dict_stock)
    print("📤 Sent:", dict_stock)
    sleep(1)

```
```bash

```
- 📤 Terminal 3 – Start Producer:
```bash
python3 producer.py
```
---

## 🧑‍🏫 Step 5: Create the Data Reader (Consumer)

### 5.1 Create a file called `consumer.py`
- ✅ Final Kafka Consumer Script to Save Messages to S3 (consumer_to_s3.py)
```python
from kafka import KafkaConsumer
from json import loads, dump
from s3fs import S3FileSystem

# ✅ Your Kafka MSK broker
KAFKA_BROKER = 'b-1.mskcluster.sv5gwy.c3.kafka.ap-south-1.amazonaws.com:9098'

# ✅ Create Kafka Consumer
consumer = KafkaConsumer(
    'MSKTutorialTopics',
    bootstrap_servers=[KAFKA_BROKER],
    security_protocol="SSL",  # keep SSL if MSK is TLS enabled
    ssl_cafile="/root/kafka_2.13-3.6.0/bin/ca.pem",
    ssl_certfile="/root/kafka_2.13-3.6.0/bin/service.cert",
    ssl_keyfile="/root/kafka_2.13-3.6.0/bin/service.key",
    value_deserializer=lambda x: loads(x.decode('utf-8'))
)

# ✅ Initialize S3 (assumes IAM role or credentials already configured)
s3 = S3FileSystem()

# ✅ Continuously consume messages and store them in S3
for count, msg in enumerate(consumer):
    s3_path = f"s3://yashdatabase/targetdata(Consume)/stock_market_{count}.json"
    print(f"Writing to {s3_path}")
    with s3.open(s3_path, 'w') as f:
        dump(msg.value, f)

```
-  📥 Terminal 4 – Start Consumer:
```bash
python3 consumer.py
```
---

---

## 🌥️ BONUS: Use AWS MSK Instead of Local Kafka (Advanced)

- Set up **MSK Cluster** on AWS
- Update `bootstrap_servers='your-msk-broker-url:9098'`
- Add **IAM Authentication** if needed
- Use **client.properties** and add IAM JAR

---

## 🏁 Done!

🎉 You just made a real-time stock price system like a pro!

---

## 🧼 Clean Up

Stop everything with `Ctrl + C` in each terminal when done.
