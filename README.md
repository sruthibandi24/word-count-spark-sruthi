
# 📊 Word Count on AWS with PySpark, Node.js Deployment, and ETL Pipeline  
**Submitted by:** Sruthi Bandi  
**Student Email:** sruthi@uncc.edu  

---

## 🚀 Project 1: Word Count on AWS EC2/LightSail using PySpark

### 🎯 Objective  
Set up PySpark on an AWS instance to count words in a text file stored in an S3 bucket.

---

### ✅ 1. Prerequisites  
Ensure you have:

- An AWS account with access to EC2/LightSail  
- An S3 bucket containing your text file  
- SSH access to the instance  

---

### ✅ 2. Set Up EC2/LightSail Instance  

Launch an Amazon Linux 2 instance and connect via SSH:

```bash
ssh -i "your-key.pem" ec2-user@your-instance-ip
```

Install Java 11:

```bash
sudo yum install java-11 -y
export JAVA_HOME=$(dirname $(dirname $(readlink -f $(which java))))
java --version
```

Increase `/tmp` size to prevent Spark errors:

```bash
sudo mount -o remount,size=2G /tmp
```

Install Python and PySpark:

```bash
sudo yum install python3-pip -y
pip install pyspark
spark-submit --version
```

---

### ✅ 3. Word Count Script  

Create `word_count.py`:

```python
from pyspark.sql import SparkSession

# AWS Credentials
AWS_ACCESS_KEY_ID = 'YOUR_ACCESS_KEY' 
AWS_SECRET_ACCESS_KEY = 'YOUR_SECRET_KEY' 

# S3 paths
S3_INPUT = 's3a://wordcountshruthi/ccwordcount1.txt'
S3_OUTPUT = 's3a://wordcountshruthi/output_folder/'

# Spark Session
spark = SparkSession.builder \
    .appName("WordCount") \
    .config("spark.jars.packages", "org.apache.hadoop:hadoop-aws:3.3.1,com.amazonaws:aws-java-sdk-bundle:1.11.901") \
    .getOrCreate()

# Hadoop S3 Configuration
hadoop_conf = spark.sparkContext._jsc.hadoopConfiguration()
hadoop_conf.set("fs.s3a.access.key", AWS_ACCESS_KEY_ID)
hadoop_conf.set("fs.s3a.secret.key", AWS_SECRET_ACCESS_KEY)

# Word Count Logic
text_file = spark.sparkContext.textFile(S3_INPUT)
counts = text_file.flatMap(lambda line: line.split()) \
                  .map(lambda word: (word, 1)) \
                  .reduceByKey(lambda a, b: a + b)
counts.saveAsTextFile(S3_OUTPUT)
spark.stop()
```

Run with:

```bash
spark-submit word_count.py
```

---

## 🧪 Project 2: Deploy a Node.js Web Server with Docker

### 🎯 Objective  
Deploy a simple Node.js server in a Docker container, push the image to Docker Hub, and run it on an EC2 instance.

---

### ✅ 1. Node.js Server  

Create a directory and initialize a project:

```bash
mkdir node-webserver && cd node-webserver
npm init -y
npm install express
```

Create `server.js`:

```javascript
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
    res.send('Hello, World! Running in Docker.');
});

app.listen(port, () => console.log(`Server running at http://localhost:${port}`));
```

---

### ✅ 2. Dockerize the Application  

Create a Dockerfile:

```dockerfile
FROM node:14
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

Build and push the image to Docker Hub:

```bash
docker build -t sruthibandi242/webserver:latest .
docker push sruthibandi242/webserver:latest
```

---

### ✅ 3. Deploy on EC2  

Install Docker:

```bash
sudo yum update -y
sudo yum install docker -y
sudo service docker start
sudo usermod -aG docker ec2-user
```

Run the container:

```bash
docker pull sruthibandi242/webserver:latest
docker run -d -p 80:3000 sruthibandi242/webserver:latest
```

Access the app at:

```
http://3.239.175.139/
```

---

## 🔗 Links  

- **GitHub Repo:** [https://github.com/sruthibandi24/word-count-spark-sruthi](https://github.com/sruthibandi24/word-count-spark-sruthi)  
- **DockerHub Repo:** [https://hub.docker.com/repository/docker/sruthibandi242/webserver/general](https://hub.docker.com/repository/docker/sruthibandi242/webserver/general)  
- **Live EC2 Web App:** [http://3.239.175.139/](http://3.239.175.139/)
