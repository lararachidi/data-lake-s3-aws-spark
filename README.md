## Summary
The purpose of this repo is to build a data lake using AWS S3 and Spark with tables designed to optimize queries on song play analysis. Data is loaded from an S3 bucket, processed into analytics tables with Spark and loaded back into S3. The pipeline is deployed using a Spark cluster on AWS.

## Datasets

There are two datasets that reside in S3:

- Song data: `s3://udacity-dend/song_data`
- Log data: `s3://udacity-dend/log_data`

### Song Dataset

The first dataset is a subset of real data from the [Million Song Dataset](https://labrosa.ee.columbia.edu/millionsong/). Each file is in JSON format and contains metadata about a song and the artist of that song. The files are partitioned by the first three letters of each song's track ID. 

For example, here is the filepath to a file in this dataset: `song_data/A/A/B/TRAABJL12903CDCF1A.json`

And below is an example of what a single song file, `TRAABJL12903CDCF1A.json`, looks like.

```{"num_songs": 1, "artist_id": "ARJIE2Y1187B994AB7", "artist_latitude": null, "artist_longitude": null, "artist_location": "", "artist_name": "Line Renaud", "song_id": "SOUPIRU12A6D4FA1E1", "title": "Der Kleine Dompfaff", "duration": 152.92036, "year": 0}```


#### Log Dataset

The second dataset consists of log files in JSON format generated by this [event simulator](https://github.com/Interana/eventsim) based on the songs in the dataset above. These simulate activity logs from a music streaming app based on specified configurations.

The log files in the dataset are partitioned by year and month. For example, here is the filepath to a file in this dataset:
`log_data/2018/11/2018-11-12-events.json`

</details>
    
## Installation

1. Set up your AWS access key and secret access key within the [dl.cfg.example](dl.cfg.example) file and rename the file as `dl.cfg`:

```config
[AWS]
AWS_ACCESS_KEY_ID=<YOUR AWS ACCESS KEY>
AWS_SECRET_ACCESS_KEY=<YOUR AWS SECRET ACCESS KEY>

[S3]
INPUT_DATA=<YOUR INPUT DATA FOLDER / BUCKET>
OUTPUT_DATA=<YOUR OUTPUT DATA FOLDER / BUCKET>
```

2. Install AWS CLI locally: link [here](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)

3. Configure your AWS profile locally: `aws configure --profile <YOUR PROFILE NAME>`
    
### Option 1: Run spark cluster locally

Within an environment configured to run Spark, run the following command from the terminal:

```bash
python etl.py
```

</details>

### Option 2: Run spark cluster on AWS

#### Start Up your EMR Cluster:

Create an EMR Cluser via AWS CLI:

```bash
aws emr create-cluster --name my_spark_cluster \ 
--use-default-roles \
--release-label emr-5.28.0 \
--instance-count <INSTANCES INCLUDING MASTER NODE e.g 3> \
--applications Name=Spark \
--ec2-attributes KeyName=<YOUR PEM KEY NAME>,SubnetId=<YOUR SUBNET ID> \
--instance-type <EC2 instance type e.g m5.xlarge>
--profile <YOUR AWS PROFILE e.g default>
```

#### Connect to Master Node from a BASH shell and update the spark-env.sh file:

1. Log onto the AWS Console and view the security group ID for the Master Node via EC2 dashboard → Security Groups service. Edit the security group to authorize inbound SSH traffic (port 22) from your local computer.

2. Connect to the EMR cluster using the SSH protocol. Obtain your EC2 IP address for the master node from the AWS Console via EC2 dashboard → Instances.

```bash
ssh -i <PATH_TO_MY_KEY_PAIR_FILE>.pem hadoop@<EC2_IP_ADDRESS><YOUR_AWS_REGION>.compute.amazonaws.com
```

3. Using sudo, append the following line to the /etc/spark/conf/spark-env.sh file:

```bash
export PYSPARK_PYTHON=/usr/bin/python3
```

#### Create a local tunnel to the EMR Spark History Server on your Linux machine:

Open up a new Bash shell and run the following command (using the proper IP for your master node):

```bash
ssh -i <PATH_TO_MY_KEY_PAIR_FILE>.pem -N -L 8157:<EC2_IP_ADDRESS>.<YOUR_AWS_REGION>.compute.amazonaws.com:18080 hadoop@<EC2_IP_ADDRESS>.<YOUR_AWS_REGION>.compute.amazonaws.com
```

- Note: This establishes a tunnel between your local port 8157 and port 18080 on the master node.

You can pick a different unused number for your local port instead.

The list of ports on the EMR side and what UIs they offer can be found [here](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-web-interfaces.html)

3. Go to localhost:8157 in a web browser on your local machine and you should see the Spark History Server UI

#### Run your Spark job

1. SFTP the dl.cfg and etl.py files to the hadoop account directory on EMR. Note: 
Secure File Transfer Protocol (SFTP), also called SSH File Transfer Protocol, is a network protocol for accessing, transferring and managing files on remote systems. SFTP allows businesses to securely transfer billing data, funds and data recovery files.

2. In your home directory on the EMR master node (/home/hadoop), run the following command:

```bash
spark-submit --master yarn etl.py
```

- Note: full path to `etl.py` is required

3. After a couple of minutes your job should show up in the Spark History Server page in your browser

- You should see the real-time logging output in your EMR bash shell window as well

</details>


## Architecture

<details open>
    <summary> Show/Hide Details</summary>

### Spark

Spark enables schema on read and acts as a distributed SQL query engine. The ETL job is significantly more efficient as a result of leveraging Spark's distributed engine.


### AWS S3

AWS S3 is used as the data lake as it provides high durability and ability to use native AWS services such as AWS EMR, AWS Glue, AWS lambda and AWS Athena. AWS S3 also supports unstructured and structured data types.

### AWS EMR

Elastic Map Reduce (EMR) is a service offered by AWS that removes the need to install Spark and its dependencies. In addition, EMR handles node categorisation by categorising secondary nodes into core and task nodes. 

</details>

## Structure

<details open>
    <summary> Show/Hide Details</summary>

* [etl.py](etl.py): spark job to load data from s3, process data with spark and load back into S3
* [dl.cfg](d.cfg): config for your AWS credentials 

</details>
