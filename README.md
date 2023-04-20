
A simulated music streaming App has grown their user base and song database and wants to move their processes and data onto the cloud. Their data resides in AWS S3, in a directory of JSON logs on user activity on the app, as well as a directory with JSON metadata on the songs in their app.

This project builds a pipeline by extracting data from AWS S3 and load them onto AWS Redhsift. A star Schema is designed with a fact table songplay and four dimensional tables for further business intelligence.

The specifc steps are as follows:

1. Pre-steps

- AWS account and IAM user

First of all, an AWS account needs to be created, and it gives a user complete access to all AWS services and resources in the account. 
Within each account, Amazon provides an Identity and Access Management (IAM) service, with which users can securely control access to AWS resources. Accordingly, we create an IAM user and attach the existing AdministratorAccess policy to it at AWS IAM service at the very beginning and save the access key ID and secret access key.

- Data warehouse parameters

As we will build a data warehouse on Redshift and load the data into it, we create a configuration file, dwh.cfg, and save data warehouse parameters and data paths to it, and then use the ConfigParser class in Python to load the data warehouse parameters from dwh.dfg.

- Create clients for IAM, EC2, S3 and Redshift

AWS services can be accessed in three ways, the AWS Management Console, the Command Line Interface (CLI), and Software Development Kits (SDKs). In this project, we use BOTO3 SDK. 

Using the access key ID and secret access key, we create clients for IAM, EC2, S3 and Redshift. * Amazon EC2 stands for Amazon Elastic Compute Cloud and provides scalable computing capacity in the AWS Cloud.

- Check out the data sources on S3

Before getting started on the pipeline, we use the S3 client created above to check out the data source on S3 and confirm the existence of the raw data.

2. Create destination data repository on AWS Redshift

- IAM role

IAM roles are a secure way to grant permissions to entities that a user trusts. In this project, we create an IAM role and attach the AmazonS3ReadOnlyAccess policy to it, so that it enables Redshift to access S3 bucket read only. Then we get its Amazon Resource Name (ARN) that specifies the IAM role for later use. 

- Redshift cluster

According to the documents of AWS, an Amazon Redshift data warehouse is a collection of computing resources called nodes, which are organized into a group called a cluster; each cluster runs an Amazon Redshift engine and contains one or more databases.

We use the Redshift client created above to create a Redshift cluster, and within the cluster we set the number of nodes at 4 and create a database with the parameters loaded in the pre-steps.

Then we get the cluster endpoint and open an incoming TCP port to access the cluster endpoint.

We use postgresql to connect to Redshift.

3. Implementation - Extract and Load

We create two staging tables, staging_events and staging_songs, in the database we created on the Redshift cluster, extract the data from the source on S3 and load the data into the staging tables using the COPY statement.

4. Implementation - Transform

One advantage of an ELT pipeline is that we can transform only the data that we need for our particular analysis. In this project, our concern is song plays. That is, we only need the data that is related to the user activities where the value of “page” is NextSong.

So, after looking into the raw data, we design a star schema for the transformation and create one fact table and four dimensional tables as the production tables.

Then we use the INSERT INTO ... SELECT statement to move the data from the staging tables to the five final production tables.

- Fact table: songplay

The fact table, songplay, records the information of each song play. It has nine columns, among which songplay_id is the primary key and is generated sequentially using the IDENTITY property, and the following eight columns are filled in with the information from the staging_events and staging_songs tables.

For the last eight columns, we first left join the staging_events table and the staging_songs table on both the titles and artist names of songs. We do so because different songs may share the same title, while one distinct artist is barely likely to have two songs with the same title. Joining the two tables on both the titles and artiest names of songs makes sure the information from the two tables for each row is about the same song. We then filter the data with the value of the page column in the staging_events table being “NextSong”, which indicates a song play.

- Dimensional table: song

This table consists of the song_id, title, artist_id, year of release and duration of each distinct song. We simply insert corresponding data from the staging_songs table into this song table.

- Dimensional table: artist

This table describes the information of the singer of each distinct song, including artist_id, name and location that are extracted from the staging_songs table.

- Dimensional table: users

This table records the information of each distinct user. It contains the user_id, name, gender and level that are imported from the staging_events table.

- Dimensional table: time

This table keeps the starting time of each song play and breaks it down into hour, day, week, month, year and weekday. Breaking down the timestamp will facilitate further analysis of the data based on time. For example, we can look into any patterns of the song plays during different hours of a day.

Then an ELT pipeline has been built. We can use the well-prepared data for business intelligence.
