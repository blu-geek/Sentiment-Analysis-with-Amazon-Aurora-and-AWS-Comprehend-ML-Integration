
# Sentiment Analysis With Amazon Aurora PostgreSQL DB and AWS Comprehend ML Integration

As a Generative AI Engineer at a large e-commerce company, you want to monitor customer feedback in real-time to improve your organization's services. They store customer reviews in an Amazon Aurora database. By integrating Aurora with Amazon Comprehend, they can automatically analyze the sentiment of each review as it is inserted into the database. This allows the company to quickly identify and respond to negative reviews, ensuring high customer satisfaction and addressing issues promptly.

# Amazon Aurora
<img width="594" alt="Screenshot 2025-01-16 at 17 01 42" src="https://github.com/user-attachments/assets/ebcb0fda-9040-41fd-8724-aaadf4c9b87a" />

# Amazon Comprehend
<img width="771" alt="Screenshot 2025-01-16 at 17 09 33" src="https://github.com/user-attachments/assets/9f067ab6-a84a-4fda-9565-6a67f2987265" />



# Activity Guide: Performing Sentiment Analysis with Amazon Aurora ML Integration

1. Create IAM Roles (IAM, Amazon S3)
- Create S3 Bucket:
Go to the S3 Console and create a bucket (e.g., amazon-reviews-pds-k21).
Uncheck "Block Public Access" and acknowledge the warning.
Upload the sample file sample_us.tsv to the bucket.
- Create Roles:
Navigate to IAM Console > Roles > Create Role.
Select RDS as the trusted entity type and choose RDS – Add Role to Database.
Attach the ComprehendReadOnly policy, name the role (e.g., AuroraComprehendRole), and create it.
Repeat the process for the second role, attaching the AmazonS3ReadOnly policy (e.g., AuroraS3Role).

2. Create Aurora Database Instance (Amazon RDS)
- Create Database:
Navigate to the RDS Console, choose Databases > Create Database.
Select Aurora (PostgreSQL-Compatible) with version 15.4.
- Configure Database:
Select Dev/Test template and set the DB instance type to db.t3.medium.
Enable public access and create a new VPC, DB Subnet Group, and Security Group.
- Attach IAM Roles:
Once the instance is available, attach the roles AuroraComprehendRole and AuroraS3Role.
Wait for the roles to show as Active.

3. Install PostgreSQL Client (pgAdmin)
- Download pgAdmin:
Go to the pgAdmin website and download the appropriate version for your OS.
- Install pgAdmin:
Follow the installation instructions.
Open pgAdmin and configure it for connecting to the Aurora database.

4. Connect to the Aurora DB Instance (pgAdmin, RDS)
- Set Up Connection:
Add a new server in pgAdmin and name it (e.g., LabServer).
Enter the DB endpoint as the host and provide the database password.
- Test Connection:
Expand the database navigation tree and open the Query Tool to execute SQL commands.

5. Query Database with Amazon Comprehend (RDS, Comprehend)
- Install ML Extensions:
Run the following in the query editor to install AWS ML extensions:
CREATE EXTENSION IF NOT EXISTS aws_ml CASCADE;
CREATE EXTENSION IF NOT EXISTS aws_s3 CASCADE;
- Create Tables and Add Data:
Create a comments table and populate it with sample data.
CREATE TABLE comments (
    comment_id SERIAL PRIMARY KEY,
    comment_text VARCHAR(255) NOT NULL
);
INSERT INTO comments (comment_text) VALUES 
('This is very useful, thank you!'),
('Awesome, I was waiting for this feature.');
- Perform Sentiment Analysis:
Run the following to analyze sentiment:
SELECT * FROM comments, aws_comprehend.detect_sentiment(comments.comment_text, 'en') AS s;
- Load S3 Data:
- Load additional data into a table from the S3 bucket using:
SELECT aws_s3.table_import_from_s3(
    'review_simple', '<schema>', '<options>', 'bucket-name', 'file-path', 'region'
);
- Update Sentiment Data:
- Use Comprehend to update sentiment and confidence scores:
UPDATE review_simple
SET scored_sentiment = s.sentiment, scored_confidence = s.confidence
FROM aws_comprehend.detect_sentiment(review_simple.review_body, 'en') AS s
WHERE scored_sentiment IS NULL;

6. Test and Summarize Results (Amazon RDS)
- Query Data:
View results of sentiment analysis:
SELECT customer_id, review_id, review_body, scored_sentiment, scored_confidence FROM review_simple;
- Summarize data:
SELECT scored_sentiment, COUNT(*) AS nReviews FROM review_simple GROUP BY scored_sentiment;

7. Clean Up Resources (Amazon RDS, IAM, S3)
- Delete Aurora Database:
Navigate to the RDS Console, select the database, and delete it.
- Delete VPC:
In the VPC Console, delete the VPC used for the database.
- Delete IAM Roles:
Remove AuroraComprehendRole and AuroraS3Role from the IAM Console.
- Delete S3 Bucket:
Empty and delete the bucket used for storing the sample data.

8. Troubleshooting (Amazon RDS, IAM, Comprehend)

- Connection Issues:
Ensure public access is enabled for the database and security group rules allow access.
- Permission Errors:
Confirm IAM roles are attached correctly to the Aurora instance.

