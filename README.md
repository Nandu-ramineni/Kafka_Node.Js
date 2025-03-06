# Kafka with Node.js

## 🚀 Introduction to Kafka

Apache Kafka is a **distributed event streaming platform** used for **high-performance data pipelines, streaming analytics, data integration, and mission-critical applications**. It is designed to handle **real-time data streams** in a **fault-tolerant, scalable, and durable** manner.

### 🛠 Key Features:
- **Publish-Subscribe Messaging:** Kafka allows multiple consumers to receive the same message.
- **High Throughput & Low Latency:** Handles a large number of messages with minimal delay.
- **Scalability:** Supports multiple partitions to distribute data efficiently.
- **Fault Tolerance:** Ensures data persistence and recovery even in case of failures.

---

## 📌 Real-Time Applications of Kafka

Kafka is widely used in industries for various real-time applications, such as:

1. **Log Aggregation & Monitoring** - Centralized logging systems like ELK Stack use Kafka to process logs in real time.
2. **E-Commerce & Online Transactions** - Platforms like Amazon, Swiggy, Zomato use Kafka for order tracking and payment processing.
3. **Fraud Detection** - Banking systems analyze transactions in real time to detect fraud.
4. **IoT Data Processing** - Smart devices stream data continuously using Kafka for processing.
5. **Social Media & Streaming Services** - Platforms like LinkedIn and Netflix use Kafka to deliver personalized recommendations.

---

## 🏗 Installation Guide

### 1️⃣ Install Kafka via Docker
If you don’t want to install Kafka manually, the easiest way is through **Docker**.

#### Step 1: Run Zookeeper & Kafka in Docker
```bash
# Start Zookeeper
docker run -d --name zookeeper -p 2181:2181 wurstmeister/zookeeper

# Start Kafka
docker run -d --name kafka -p 9092:9092 \
-e KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 \
-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://localhost:9092 \
-e KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1 \
--network=host wurstmeister/kafka
```

#### Step 2: Verify Kafka is Running
```bash
docker ps
```

### 2️⃣ Install Kafka Manually (Without Docker)
If you prefer a manual setup, follow these steps:

#### Step 1: Download Kafka
```bash
wget https://downloads.apache.org/kafka/3.0.0/kafka_2.13-3.0.0.tgz
```

#### Step 2: Extract & Navigate
```bash
tar -xvzf kafka_2.13-3.0.0.tgz
cd kafka_2.13-3.0.0
```

#### Step 3: Start Zookeeper & Kafka
```bash
# Start Zookeeper
bin/zookeeper-server-start.sh config/zookeeper.properties

# Start Kafka
bin/kafka-server-start.sh config/server.properties
```

#### Step 4: Verify Kafka Installation
```bash
bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

---

## 💻 Kafka with Node.js (Producer & Consumer)

### **1️⃣ Producer (producer.js)**
This producer reads user input and sends messages to Kafka based on the partition logic.

```javascript
import { Kafka } from "kafkajs";
import readline from "readline";

const kafka = new Kafka({
    clientId: "my-app",
    brokers: ["localhost:9092"]
});

const producer = kafka.producer();

const produceMessage = async () => {
    await producer.connect();
    console.log("Producer Connected");

    const rl = readline.createInterface({ input: process.stdin, output: process.stdout });
    rl.setPrompt('> ');
    rl.prompt();
    
    rl.on('line', async function(line) {
        const [riderName, location] = line.split(' ');
        await producer.send({
            topic: 'test-group-1',
            messages: [{
                partition: location.toLowerCase() === 'south' ? 0 : 1,
                key: 'Location Updates',
                value: JSON.stringify({ name: riderName, location })
            }]
        });
    }).on('close', async () => {
        await producer.disconnect();
    });
};

produceMessage().catch(console.error);
```

---

### **2️⃣ Consumer (consumer.js)**
This consumer listens to messages from Kafka.

```javascript
import { Kafka } from "kafkajs";

const group = process.argv[2];
const kafka = new Kafka({
    clientId: "my-app",
    brokers: ["localhost:9092"]
});

const consumer = kafka.consumer({ groupId: group });

const consumeMessage = async () => {
    await consumer.connect();
    await consumer.subscribe({ topic: "test-group-1", fromBeginning: true });
    
    await consumer.run({
        eachMessage: async ({ topic, partition, message }) => {
            console.log(
                `${group}: [${topic}] PART:${partition}:`,
                message.value.toString()
            );
        }
    });
};

consumeMessage().catch(console.error);
```

### **3️⃣ Running the Code**

#### **Start the Producer**
```bash
node producer.js
```
Now, type a message like:
```bash
bill south
tom north
```

#### **Start Two Consumers in Different Terminals**
```bash
node consumer.js user-1
```
```bash
node consumer.js user-2
```
Each consumer will receive messages based on the assigned partitions.

---

## 🎯 Conclusion
Kafka is a powerful tool for real-time streaming applications. By integrating Kafka with Node.js, we can build **scalable and high-performance** event-driven applications. 🚀


