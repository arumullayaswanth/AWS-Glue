# How to create MSK Cluster on AWS | Publish Messages to Kafka Topic | Run Kafka on AWS

## ☁️ AWS MSK Kafka End-to-End Setup (With IAM Auth & EC2 Client)

## ✅ Prerequisites
- AWS Account
- IAM Role with MSK permissions
- Java 11 on EC2 Client
- Kafka 3.6.0 binaries
- IAM-based authentication for MSK

---
## ✅ 1. Create Networking Setup for MSK and EC2

---

## 🧱 Step 1: Create a VPC

1. Go to **AWS Console** → **VPC Dashboard**
2. Click on **“Create VPC”**
3. Choose **VPC only**
4. Set:
   - **Name**: `msk-vpc`
   - **IPv4 CIDR block**: `10.0.0.0/16`
   - Leave rest as default
5. Click **Create VPC**

---

## 🌐 Step 2: Create Public Subnets

1. Go to **Subnets** in the left sidebar
2. Click **Create subnet**
3. Select:
   - **VPC**: `msk-vpc` (created earlier)
   - Add **2 subnets** in different AZs, e.g.:
     - `10.0.1.0/24` in `us-east-1a`
     - `10.0.2.0/24` in `us-east-1b`
4. Set names like:
   - `msk-subnet-1`
   - `msk-subnet-2`
5. Click **Create subnet**

---

## 🌍 Step 3: Enable Auto-Assign Public IP (optional for EC2)

1. Go to **Subnets**
2. Select the subnet for EC2
3. Click **Actions → Edit subnet settings**
4. Turn ON **Auto-assign public IPv4 address**

---

## 🔐 Step 4: Create Security Groups

### 🔸 A. Create Security Group for MSK Cluster

1. Go to **EC2 → Security Groups**
2. Click **Create Security Group**
   - **Name**: `MSK-SG`
   - **VPC**: `msk-vpc`
3. Leave inbound rules **empty for now**
4. Note the **Group ID** (used in next step)

---

### 🔸 B. Create Security Group for EC2 Client

1. Again, click **Create Security Group**
   - **Name**: `EC2-Client-SG`
   - **VPC**: `msk-vpc`
2. Under **Inbound Rules**, add:
   - Type: `SSH`
   - Port: `22`
   - Source: `My IP`
3. Save and **note the Group ID**

---

## 🔁 Step 5: Allow MSK to Accept from EC2 Client

1. Go to **MSK-SG** (Security Group of the Kafka cluster)
2. Click **Edit inbound rules**
3. Add a rule:
   - Type: **All traffic**
   - Source: **Security Group ID of EC2-Client-SG**
4. Save

---

✅ **Networking setup complete!**


---

## 2. 🔐 IAM Role for Kafka Access

### Trust Policy
Attach to the **EC2 instance**.

### Permissions Policy
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "kafka-cluster:Connect",
        "kafka-cluster:AlterCluster",
        "kafka-cluster:DescribeCluster"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "kafka-cluster:*Topic*",
        "kafka-cluster:WriteData",
        "kafka-cluster:ReadData"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "kafka-cluster:AlterGroup",
        "kafka-cluster:DescribeGroup"
      ],
      "Resource": "*"
    }
  ]
}
```

---
# Kafka

## 3. 🏗️ Create MSK Cluster (SASL/IAM Auth)

1. Go to **MSK → Create Cluster**
   - Creation method : `Quick create`
   - Cluster name : `MSK-Cluster`
   - General cluster properties 
        - Cluster type : `Provisioned`
        - Apache Kafka version : `3.8.x (recommended)`
        - Metadata mode : `ZooKeeper`
        - Broker type : `Standard brokers`
        - Broker size : `kafka.t3.small` **it's only for demo**
        - Storage : `100 Gib` **it's only for demo**
  - create **Custom**     
---

## 4. 🖥 Launch EC2 Client

1. Go to **EC2 → Launch instance**
      - name : `kafka`
      - Amazon Machine Image (AMI) : `Amazon Linux 2023 kernel-6.1 AMI`
      - Instance type : `t2.micro`
      - Attach **IAM role** and **EC2 Client SG**
      - lanuch instance
2. connect instance and Configure Kafka on EC2
---

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
      
```bash
bin/kafka-topics.sh --create --bootstrap-server <bootstrapServerString> --command-config ./client.properties --replication-factor 3 --partitions 1 --topic MSKTutorialTopic

# replace <bootstrapServerString>
bin/kafka-topics.sh \
--create \
--bootstrap-server b-1.mskcluster.sv5gwy.c3.kafka.ap-south-1.amazonaws.com:9098 \
--command-config /root/kafka_2.13-3.6.0/bin/client.properties \
--replication-factor 3 \
--partitions 1 \
--topic MSKTutorialTopic

```

---


### step:8
  - connect instance in one tap and runthis commands
## 8.1:  📤 Produce Messages
```bash
bin/kafka-console-producer.sh --broker-list <bootstrapServerString> --producer.config /root/kafka_2.13-3.6.0/bin/client.properties --topic MSKTutorialTopic

# replace <bootstrapServerString>

bin/kafka-console-producer.sh \
  --broker-list b-1.mskcluster.sv5gwy.c3.kafka.ap-south-1.amazonaws.com:9098 \
  --producer.config /root/kafka_2.13-3.6.0/bin/client.properties \
  --topic MSKTutorialTopic


```

Type messages and press Enter.
  - hi
  - hello 
  - hello
  - this is kafka
  - msk is working!



---

## 9. 📥 Consume Messages
```bash
bin/kafka-console-consumer.sh --bootstrap-server <bootstrapServerString> --consumer.config /root/kafka_2.13-3.6.0/bin/client.properties --topic MSKTutorialTopic --from-beginning


# replace <bootstrapServerString>

bin/kafka-console-consumer.sh \
--bootstrap-server b-1.mskcluster.sv5gwy.c3.kafka.ap-south-1.amazonaws.com:9098 \
--consumer.config /root/kafka_2.13-3.6.0/bin/client.properties \
--topic MSKTutorialTopic \
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

## 🧹 Cleanup
- Terminate EC2 Instance
- Delete MSK Cluster
- Remove Security Groups and VPC if no longer needed

---
