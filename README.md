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

### **Kafka Architecture and Working**  

## **Kafka Architecture**
Kafka's architecture consists of the following key components:

1. **Producer:** Sends messages (events) to Kafka topics.
2. **Topic:** A category in Kafka where messages are stored. Topics can have multiple partitions.
3. **Partition:** Each topic is split into multiple partitions for parallel processing and scalability.
4. **Broker:** A Kafka server that stores data and handles requests. Kafka runs as a cluster of brokers.
5. **Zookeeper:** Manages metadata, leader election, and coordination between brokers.
6. **Consumer:** Reads messages from Kafka topics.
7. **Consumer Group:** Multiple consumers working together to read messages from a topic, ensuring load balancing.

---

## **How Kafka Works?**  

### **1. Producing Messages**  
- Producers send messages to a Kafka **Topic**.
- Messages are **appended** to a topic's partitions.
- Producers can use **keys** to send messages to a specific partition.

### **2. Partitioning**  
- Topics are divided into **partitions** for scalability.
- Each partition is stored and replicated across multiple brokers.
- Messages in partitions are ordered but **Kafka does not guarantee global order** across partitions.

### **3. Storing Messages**  
- Kafka stores messages in a **log format**.
- Messages are retained based on time or log size (configurable).
- Kafka follows a **distributed commit log** model for fault tolerance.

### **4. Consuming Messages**  
- Consumers **subscribe** to a topic and read messages in **order**.
- Consumer groups ensure **parallel processing** by distributing partitions among consumers.
- Kafka keeps track of the last read message using an **offset**.

### **5. Scaling and Fault Tolerance**  
- **Multiple brokers** handle load balancing.
- **Replication** ensures data availability.
- **Consumer Groups** enable parallel processing.

---

## **Kafka Architecture Diagram**
```
              ┌───────────────┐
              │   Producer    │
              └──────┬────────┘
                     │
      ┌──────────────▼──────────────┐
      │         Kafka Cluster        │
      │  ┌────────┬────────┬───────┐ │
      │  │Broker 1│Broker 2│Broker3│ │
      │  └────┬───┴────┬───┴──────┘ │
      │       │        │            │
      │  ┌────▼───┐  ┌─▼───┐  ┌──▼─┐ │
      │  │Partition│  │Partition│  │Partition│ │
      │  └────────┘  └────────┘  └────────┘ │
      └──────────────▲──────────────┘
                     │
             ┌───────▼──────┐
             │   Consumer   │
             └──────────────┘
```

---

## **Use Case: How Your Kafka Code Works**
1. **Producer (`producer.js`)**  
   - Reads input from the terminal.
   - Sends messages to Kafka with partitioning logic.
   - Uses a **key-based** partitioning approach.

2. **Consumer (`consumer.js`)**  
   - Listens to Kafka topics.
   - Uses a **consumer group** to balance the load.
   - Reads messages in real-time.

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


