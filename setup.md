# Setup Guide for Spotify ETL Project on AWS

This guide will walk you through the setup of a fully automated Spotify ETL pipeline on AWS, where data is fetched from the Spotify API, transformed, and stored in AWS services such as S3, Glue, and Athena. 

## Prerequisites

- AWS account with necessary permissions to create Lambda functions, Glue Crawlers, Athena, EventBridge, and S3 buckets.
- Spotify Developer account for accessing the Spotify API.

---

### Step 1: Set Up a Spotify Developer Account

1. **Create an Account on Spotify Developer Portal**  
   Go to [Spotify Developer Dashboard](https://developer.spotify.com/dashboard/applications) and sign in or create an account.

2. **Create a Spotify App**  
   - Click on **"Create an App"** to get your **Client ID** and **Client Secret**.
   - Save the **Client ID** and **Client Secret**, as they will be needed in the Lambda function for authentication.
   - Define the redirect URI, which you can set to `http://localhost:8888/callback`.

3. **Get Access Tokens**  
   - Use the **Client ID** and **Client Secret** to get an **OAuth token**.  
     You can follow the [Spotify Authentication Guide](https://developer.spotify.com/documentation/general/guides/authorization-guide/) to implement OAuth and get the access token.

---

### Step 2: Set Up AWS Services

1. **Create an S3 Bucket**  
   Create two S3 buckets:  
   - One for **Raw data** (to store the Spotify API data).  
   - Another for **Transformed data** (after the data is cleaned).

   Example:
   - `spotify-etl-project-raw`
   - `spotify-etl-project-transformed`

2. **Create Lambda Functions**  
   Create two AWS Lambda functions:
   - **Extraction Lambda**: This Lambda will call the Spotify API and store the raw data in the S3 bucket at regular intervals (15 minutes).
   - **Transformation Lambda**: This Lambda will be triggered when a new JSON file is uploaded to the S3 bucket, process the file, and store the transformed data back to the S3 bucket.

   Follow these steps to set up the Lambda functions:
   - Go to the Lambda service in AWS.
   - Create a new Lambda function for **Extraction**.
     - Use Python as the runtime.
     - Add the necessary Spotify API code using the **Client ID** and **Client Secret** for authentication.
     - Store the extracted data in the **Raw data S3 bucket**.

     **Image for Extraction Lambda Setup:**
     ![Extraction Lambda](https://github.com/anupamjha977/Spotify_Etl_Project/blob/main/images/extraction%20lambda.JPG)

   - Create another Lambda function for **Transformation**.
     - This Lambda will be triggered by an S3 event when a new JSON file is uploaded.
     - It will clean and process the data and save the results into the **Transformed data S3 bucket**.

     **Image for Transformation Lambda Setup:**
     ![Transformation Lambda](https://github.com/anupamjha977/Spotify_Etl_Project/blob/main/images/transformation.JPG)

3. **Set Up EventBridge**  
   Use **Amazon EventBridge** to schedule the **Extraction Lambda** to run every 15 minutes. This will trigger the extraction process from Spotify and store the raw data in the S3 bucket.

---

### Step 3: Set Up Glue Crawler

1. **Create Glue Crawler**  
   Create a **Glue Crawler** to automatically detect and catalog the schema of the transformed data stored in the S3 bucket.

   - Go to **AWS Glue Console**.
   - Create a **Crawler** and set the source as the **Transformed data S3 bucket**.
   - Set the output to the **Glue Data Catalog**.

   **Image for Glue Crawler Setup:**
   ![Glue Crawler](https://github.com/anupamjha977/Spotify_Etl_Project/blob/main/images/crawler.JPG)

2. **Run the Crawler**  
   - The Glue Crawler will scan the transformed data and create or update the metadata in the Glue Data Catalog.
   - This will allow for easy querying through **Amazon Athena**.

---

### Step 4: Set Up Athena for Querying Data

1. **Create Athena Database**  
   - Go to the **Athena Console** and create a new database.
   - Use the Glue Data Catalog as the source for querying the transformed data.

2. **Run Queries**  
   - Once the Glue Crawler has populated the catalog, you can query the data in Athena using SQL.
   - Here is an example query to get data about artists:

   ```sql
   SELECT artist_id, artist_name, external_url
   FROM spotify_data
   LIMIT 10;

