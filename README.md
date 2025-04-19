## Realtime-Data-Engineering-Project-With-Airflow-Kafka-Spark-Cassandra-and-Postgres

## Project Overwiew
Data engineering is an essential part of the modern data ecosystem, enabling organizations to make sense of large volumes of data. Creating a robust, scalable, and fault-tolerant data pipeline is a complex task that involves multiple tools and technologies. This is an end-to-end data engineering project that uses Docker, Apache Airflow, Apache Kafka, Apache Spark, Apache Cassandra, and PostgreSQL.

---

## Architecture Overview
- **Data Ingestion:** Raw data is ingested into the system using Kafka. The data can come from various sources like IoT devices, user activity logs, etc.
- **Data Processing:** Airflow schedules Spark jobs to process the raw data. The processed data can either be aggregated, filtered, or transformed based on the business logic.
- **Data Storage:** The processed data is stored in either Cassandra for NoSQL needs or PostgreSQL for relational data storage.
- **Orchestration:** Docker containers encapsulate each component of the architecture, ensuring isolation and ease of deployment.

---
## Setup and Workflow

![System Architecture](https://github.com/airscholar/e2e-data-engineering/blob/main/Data%20engineering%20architecture.png)

---
The pipeline is designed with the following components:

- **Data Source**: We use `randomuser.me` API to generate random user data for our pipeline.
- **Apache Airflow**: Responsible for orchestrating the pipeline and storing fetched data in a PostgreSQL database.
- **Apache Kafka and Zookeeper**: Used for streaming data from PostgreSQL to the processing engine.
- **Control Center and Schema Registry**: Helps in monitoring and schema management of our Kafka streams.
- **Apache Spark**: For data processing with its master and worker nodes.
- **Cassandra**: Where the processed data will be stored.

---
## Step-by-Step Guide
### **Step 1:** Folder Structure and Installation
The folder structure for this project is 2 folders and 5 files with the structure and names below:


![image](https://github.com/user-attachments/assets/518b635b-f2ae-4ced-af0a-f91e9d60c983)

And the *requirements.txt* file with the required dependencies are as shown below. Run **pip install -r requirements.txt** to install the dependencies.

**Entrypoint.sh file**
The *entrypoint.sh* file contains the commands to be executed once the attached container has been initialised. To make it work as expected, it is advised to **run chmod +x scripts/entrypoint.sh** in the root directory to convert the script to executable.
![image](https://github.com/user-attachments/assets/fee9db01-f4c5-4a10-8b33-e58e45aca554)

---

### **Step 2:** Setting up Docker Containers

- Firstly, we use Docker Compose to spin up containers for the project. The **dockerfile.yml** below creates the following containers:

- **Zookeeper**: Zookeeper acts as a distributed configuration and synchronization service. It’s particularly vital for Apache Kafka to manage its distributed nature.
- **Apache Kafka Broker**: Kafka Broker handles the storage, retrieval, and transfer of messages (records). It’s a part of the Kafka distributed streaming platform.
- **Schema Registry**: The Schema Registry provides a serving layer for your metadata. It is a RESTful interface for storing and retrieving Avro schemas which is particularly useful when your Kafka streams need to understand the schema of the records.
- **Control Center**: This is a web-based interface for managing and monitoring your Kafka environments. It provides features like data inspection, topic creation, and setting up Kafka Connect.
- **Spark Master**: Spark Master is the point of entry to any Spark functionality. It is responsible for distributing work across the Spark Cluster.
- **Spark Worker**: Spark Worker is responsible for executing the tasks that the master node assigns it and returning the computed results.
- **Cassandra DB**: Cassandra is a NoSQL database designed to handle large amounts of data across many nodes without any single point of failure. It’s ideal for high-velocity data like streaming data.
- **Apache Airflow Webserver**: Apache Airflow’s web server is the interface where you will define and monitor your workflows (DAGs). Airflow is commonly used for orchestrating complex ETL tasks.
- **Scheduler**: The Scheduler in Airflow triggers the tasks and constructs the data pipelines. It ensures tasks are executed at the right time or when triggered by other tasks.
- **PostgresDB**: PostgreSQL is a relational database. It’s used here as the metadata database for Apache Airflow and can also serve as a general-purpose data store.

- Below is the content of the **dockerfile.yml** used to orchestrate the spinning up process.

Attention: The **AIRFLOW__WEBSERVER__SECRET_KEY** must be the same for the webserver and the scheduler for the containers to work effectively.

![image](https://github.com/user-attachments/assets/cacac1fe-b466-4f42-a85b-c8d131a09b19)


To spin the containers up, run the script below:

![image](https://github.com/user-attachments/assets/c522786d-e7d5-4536-8e29-ccf3271c42e4)



![image](https://github.com/user-attachments/assets/3d5a182c-3e2b-4a00-809b-54f7e4104bea)

---
### Step 3: Streaming Data into Apache Kafka
Once the docker containers are up and running, create a new a file in the dags directory **(stream_kafka.py)**, here is it’s content:

![image](https://github.com/user-attachments/assets/2c0bdb20-8a8e-461a-94c3-3cf4047742ea)

Goto the *Airflow UI* on **localhost:8080**, and run the user_automation dag (the play button in the actions section).

![image](https://github.com/user-attachments/assets/621431e3-f209-461e-9b23-85463b5433ae)


To see the data streams on the **control center UI**, goto localhost:9021

![Control Center Landing Page](https://github.com/user-attachments/assets/bca43fd0-0c80-4434-9370-85cf9cd3f5de)



![User Created Topic](https://github.com/user-attachments/assets/86c18302-ad90-45b2-9844-7d5cec1a2d21)



![User Created Topic Data](https://github.com/user-attachments/assets/6d26a581-4de0-402f-b598-93898023d604)

---

### Step 4: Streaming Data into Cassandra
In the root directory, create a new file called **spark_stream.py** and add the following content to it.
![image](https://github.com/user-attachments/assets/46fd21e2-bef9-4510-b1dc-da304f5aaa67)

![image](https://github.com/user-attachments/assets/c0324a10-7664-4317-a0f9-623c3dbc2a9a)

![image](https://github.com/user-attachments/assets/277a873b-d6ab-41c0-af10-b51ab284e2e0)

Then run this script in your root directory to submit the job to your spark cluster:

![image](https://github.com/user-attachments/assets/e164876b-83a8-4c7d-9c2b-ae1e07f074e6)

Goto **localhost:9090** you should see the spark master and worker setup and the worker state is *ALIVE*.

![image](https://github.com/user-attachments/assets/e13268b2-ac2b-4331-abf6-387ab2baf871)

Now, if you want to see the data on cassandra, you need to connect to the cluster using:

![image](https://github.com/user-attachments/assets/ca998325-ccce-44e8-852a-343217de206e)



![image](https://github.com/user-attachments/assets/4e6b9987-4f69-4862-b6a9-0ad8862d0d8f)

When you run **DESCRIBE spark_streams.created_users;** you should see the keyspace and table created successfully

![image](https://github.com/user-attachments/assets/de61c382-c4b2-421b-998b-d7a14accf01e)


To get the data streamed into cassandra, run:

**SELECT * FROM spark_streams.created_users;**

![image](https://github.com/user-attachments/assets/8f80ed73-15f5-4d7f-aa14-5546a0f8be27)

---


























