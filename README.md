Big_Project: Muhammad Reza Adi Kusuma
02-01-2023

### Problem Statement

Cilvest, a burgeoning investment management platform currently expanding its operations, aims to collect data from Twitter to gauge the extent of public interest in investment opportunities within Indonesia. Cilvest seeks to assess the demand for several investment instruments—specifically, stocks, mutual funds, and Islamic bonds (sukuk). The analysis will focus on the volume of discussions related to these aforementioned investment instruments. As a startup in its developmental phase, Cilvest prioritizes a cost-effective data infrastructure. The desired approach involves streaming data ingestion, followed by processing and loading the data into Elasticsearch, Kibana, and BigQuery.

### Data Flow Diagram

[![workflow-big-project.png](https://i.postimg.cc/Mp3xRTtc/workflow-big-project.png)](https://postimg.cc/FYS2XhMm)

#### Infrastructure Design

In this project, the data infrastructure will be established on the Google Cloud Platform (GCP), leveraging 4 key tools:

1. Compute Engine: Utilized for continuous streaming data ingestion.
2. Cloud Storage: Serving as the data lake for storing incoming data.
3. Dataproc: Employed for batch processing using Spark.
4. BigQuery: Functioning as both the data warehouse and data mart.

To scrape Twitter data, the Twitter API along with the Tweepy module will be used. Streaming will be conducted through Apache Kafka, where Python-based producer and consumer programs will be developed. Kafka will operate continuously on a Compute Engine Virtual Machine, facilitating the continuous ingestion of data. The raw data ingested via Kafka will be stored in Cloud Storage.

For the intended analysis, the data stored in Cloud Storage will undergo transformation using Pyspark executed on Dataproc. The transformation process will occur periodically. To optimize cost-efficiency, the Dataproc cluster will be instantiated before Spark execution and subsequently terminated upon completion. The transformed data will be stored in BigQuery, Elasticsearch, and Kibana.

Following data storage in BigQuery, data mart tables will be created using Python. This program will run on a Compute Engine Virtual Machine immediately after Dataproc termination. The data mart will then be visualized periodically using Looker Studio.

#### Why this Design Is Chosen

The rationale behind the design of this data infrastructure is to address Cilvest's business inquiries regarding the extent of public demand in Indonesia concerning stocks, mutual funds, and sukuk, while considering cost efficiency.

Furthermore, the infrastructure design aligns with the nature of the data to be analyzed. Given the analysis will focus on Indonesian public tweets, the infrastructure is tailored to enable real-time data retrieval 24/7. However, practical constraints may cause intermittent scraping due to limitations imposed by Twitter. Data stored in Cloud Storage remains in raw form to facilitate diverse analyses in the future.

During the data transformation process, the selected attributes are as follows:

1. Tweet ID: Serving as a unique identifier for each data entry.
2. Tweet creation time: Capturing the timestamp of when the tweet was created.
3. Username: Identifying the account that posted the tweet.
4. Tweet text: The actual content of the tweet, intended for cleaning and analysis.
5. Language: Employed to filter out tweets from non-Indonesian users, as there isn't a definitive attribute indicating location.

After a four-day trial period, an estimated influx of approximately 1500-2000 unique tweets per day is projected. Each raw data entry averages about 10KB in size, amounting to roughly 15-20 MB per day/450-600 MB per month/5.4-7.2 GB per year. Given the relatively modest data size, utilizing a Compute Engine with limited RAM capacity is sufficient. This decision is pivotal considering that the machine type significantly impacts costs, and the Compute Engine Virtual Machine must be operational continuously. Moreover, the size of the cleaned data is notably small, estimated at around 400 KB per day/12 MB per month/144 MB per year.

Apache Kafka is chosen for streaming data ingestion due to several advantages, including its open-source nature, utilization of a publish-subscribe system, fault tolerance, and horizontal scalability. Kafka is deployed on a Compute Engine Virtual Machine to control costs.

Elasticsearch is employed as the search engine for user queries, selected for its open-source framework and exceptional search capabilities. It runs on a Compute Engine Virtual Machine to ensure immediate availability whenever required.

Data transformation is executed using Spark on Dataproc due to its robustness, distributed system architecture, elimination of the need for Spark installation, and avoidance of additional load on the Compute Engine. Cost implications of Dataproc, which tend to be high, are mitigated by creating a Dataproc instance solely when needed, estimated to operate for approximately 5 minutes per day. Thus it became negligible.

BigQuery serves as both the data warehouse and data mart due to its serverless nature, cost-effectiveness, and ease of data transfer with other GCP products. Cloud Storage serves as data lake due to the same reasons as BigQuery.

Looker Studio is chosen for dashboard creation due to its seamless integration with BigQuery, auto-update functionality, cost-efficiency, and its ability to craft interactive and visually appealing dashboards.

### Virtual Machines Specification

#### Compute Engine

Name: Big_project
Region/Zone: us-central1-c
Machine type: e2-medium (2vCPU, 4 GB RAM)
Boot disk type: balance persistent disk
Disk size: 10GB
OS: Ubuntu 20.04 LTS
Python: 3.8.10
Zoo-keeper: 3.7.0
Kafka: 2.8.2
Elasticsearch: 8.5.2
Kibana: 8.5.2

#### Dataproc

Model: dataproc on compute engine
Name: Big_projects
Region/Zone: us-central1-c
OS: Ubuntu 18.04.06 LTS
Hadoop: 3.2
Spark: 3.1.3
Python: 3.8.15

##### Master Node

N = 1
Machine type: e2-standard-2 (2vCPU, 8 GB RAM)
Primary disk type: standard persistent disk
Disk size: 10GB

##### Worker Node

N = 3
Machine type: e2-standard-2 (2vCPU, 8 GB RAM)
Primary disk type: standard persistent disk
Disk size: 10GB

### How to run Zookeeper and Kafka in the Background:

1. In the SSH window, enter the following command to start Zookeeper in the background:
   > . apache-zookeeper-3.7.0-bin/bin/zkServer.sh start
2. Once Zookeeper is successfully running, initiate Kafka by executing the following commands:
   > cd kafka_2.13-2.8.2/
   > nohup bin/kafka-server-start.sh config/server.properties &

### How to run Elasticsearch and Kibana:

1. Open the SSH Compute Engine.
2. In the SSH window, to start Elasticsearch, enter the following commands:
   > sudo systemctl daemon-reload
   > sudo systemctl start elasticsearch
3. On the Compute Engine page, copy the external IP of the VM in use.
4. Open a new tab in your browser.
5. Enter external.IP.VM:9200 in the URL bar (e.g., 35.205.154.5:9200).
6. In the previous SSH window, type the following command to start Kibana:
   > sudo /bin/systemctl start kibana.service
7. Open a new tab in your browser.
8. Enter external.IP.VM:5601 in the URL bar (e.g., 35.205.154.5:5601).
9. Please note that the external IP for each Virtual Machine changes every time it is restarted.

### Running prd_big_project.py

The provided script functions as a Kafka producer, responsible for streaming Twitter data into a specific topic. To prepare before executing this script on the Compute Engine Virtual Machine, here are the necessary steps:

1. Upload the file spark_big_project.py to the Compute Engine Virtual Machine.
2. Obtain a Twitter developer account if not already acquired; registration can be done at https://developer.twitter.com/.
3. Ensure Zookeeper and Kafka are running beforehand.
4. Create the topic "investasi" if it doesn't exist yet. Execute the following commands in the SSH window:
   > cd kafka_2.13-2.8.2/
   > bin/kafka-topics.sh --zookeeper localhost:2181 --create --topic investasi --partitions 1 --replication-factor 1
5. Prepare the file 'twitterAPISulisB.json', containing personal Twitter API keys and tokens, saved in the Compute Engine Virtual Machine.
6. Activate the virtual environment "venvbigproject" by typing the following command in the SSH window:
   > . venvbigproject/bin/activate
7. Install the necessary modules, tweepy and kafka-python, within the virtual environment by typing these commands in the SSH window:
   > pip install tweepy
   > pip install kafka-python
8. To execute this script in the background, enter the following command in the SSH window:
   > nohup python prd_big_project.py &

### Running csm_big_project.py

Here are the steps to prepare before executing the Kafka consumer script on the Compute Engine Virtual Machine. the setup will be in place for the Kafka consumer script to retrieve data from the topic and stream it to Cloud Storage:

1. Upload the file spark_big_project.py to the Compute Engine Virtual Machine.
2. Create a service account with admin access to Cloud Storage and store the service account on the Compute Engine.
3. Ensure Zookeeper and Kafka are running.
4. Create the topic "investasi" if it doesn't exist yet.
5. Activate the virtual environment "venvbigproject".
6. Install the kafka-python module within the virtual environment.
7. To execute this script in the background, enter the following command in the SSH window:
   > nohup python csm_big_project.py &

### Running spark_big_project.py

Here's a breakdown of the steps required before running the script on the Dataproc master node:

1. Create a Dataproc cluster.
2. Ensure Elasticsearch and Kibana are running.
3. Upload the file spark_big_project.py to the master node of the Dataproc cluster.
4. Create a service account with admin access to Cloud Storage and BigQuery, and store the service account on the master node of the Dataproc cluster.
5. To execute the script, use the following command in the SSH window:
   > spark-submit --packages org.elasticsearch:elasticsearch-spark-30_2.12:8.5.2,com.google.cloud.spark:spark-bigquery-with-dependencies_2.12:0.27.1 spark_big_project.py

Once the script successfully runs, remove the Dataproc cluster. This process is repeated daily.

This series of steps will ensure the execution of the script that retrieves all data from Cloud Storage on the former day, performs transformation, and sends the results to Elasticsearch and BigQuery, all within the Dataproc environment.

### Running big_project_datamart.py

Here's the preparation checklist before executing the script on the Compute Engine:

1. Create a service account with admin permissions for BigQuery and store it on the Compute Engine.
2. Successfully run spark_big_project.py.
3. Upload the file spark_big_project.py to the Compute Engine.
4. To execute the script in the background, type the following command in the SSH window:
   > python big_project_datamart.py

These steps will set up the necessary prerequisites to run the script that generates a data mart from the data warehouse in BigQuery on the Compute Engine.

### Data Visualization

#### Elasticsearch:

Here are the steps to view data in Elasticsearch using Kibana:

1. Open Kibana in your web browser.
2. Navigate to the menu and select "Management."
3. Choose "Dev Tools" from the options.
4. In the console, type (and once ready, press Ctrl+Enter):
   ```json
   get covid_tweets_project5/_search
   ```
   This command fetches data from the covid_tweets_project5 index in Elasticsearch and displays it in Kibana's Dev Tools.

### Looker Studio

1. If a dashboard hasn't been created yet, start by clicking on "Create a new report" or a similar option.
2. Choose the Google Connector, then select BigQuery as the data source.
3. Pick the project, dataset, and table you want to visualize within the dashboard.
4. Begin creating the dashboard, adding visual elements, charts, or data representations based on your selected data.
5. The data will automatically update at specified intervals or based on the settings configured within the dashboard or the data visualization tool.

### Budget

The estimated annual costs for the infrastructure design you've outlined are as follows:

Virtual Machine Compute Engine: Rp 384,473.67
Virtual Machine Dataproc (master): Rp 3,160.06
Virtual Machine Dataproc (worker): Rp 9,480.17
Cloud Storage: Rp 805.91
BigQuery: Rp 0.00
Dataproc: Rp 3,772.56
Persistent Disk Compute Engine: Rp 15,719.00
Persistent Disk Dataproc: Rp 103.12
Totaling: Rp 417,514.48 per month / Rp 5,010,173.76 per year

Adjustments in resource usage, instance types, or configurations might influence these estimations.

### Recommendations for Developing Data Infrastructure

Using Apache Airflow for scheduling tasks like creating/deleting Dataproc instances, Spark-based transformations, and data mart generation can significantly streamline the workflow. Running Apache Airflow on a Compute Engine VM offers flexibility and control over task management. It should be noted that implementing this might increase the cost.

Replacing Kafka with Pub/Sub can be a feasible alternative to reduce Compute Engine load while ensuring efficient streaming data ingestion. Pub/Sub's scalability and managed service features align well with dynamic data volume needs.

Customizing Compute Engine specifications and storage capacity is a smart move, especially for handling anticipated data growth. This adaptability ensures the infrastructure remains capable of managing increased data volumes in the future.
