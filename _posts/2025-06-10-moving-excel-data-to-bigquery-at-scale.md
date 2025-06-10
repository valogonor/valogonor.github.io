---
title: Moving Excel Data to BigQuery at Scale: An End-to-End ETL Project
---

## Introduction

Excel files are still everywhere—in finance departments, HR teams, academic research, and nonprofits. But turning that data into something useful for analysis often means tedious, manual work: cleaning, transforming, and importing it into analytical tools. I wanted to solve this problem more elegantly.

In this project, I built an automated ETL (Extract, Transform, Load) pipeline using Python and Google Cloud. The workflow uploads Excel files to Google Cloud Storage (GCS), processes and cleans the data using a Cloud Run function with pandas, and loads the final dataset into Google BigQuery for analysis. This post walks through how I designed and built it—and what I learned along the way.

## The Problem and Why It Matters

Organizations often receive important data in Excel format—from vendors, internal teams, or public sources. Manually uploading and cleaning these files is time-consuming and error-prone. By building an automated pipeline, I can reduce this overhead and ensure the data lands cleanly in BigQuery, ready for SQL analysis or visualization.

## Architecture Overview

Here's the high-level structure of the pipeline:

```
Excel Files → Google Cloud Storage → Cloud Run Cleaning Function → BigQuery
```

Each step is modular and built for scalability. Google Cloud services like GCS, Cloud Run, and BigQuery are serverless and integrate well with Python, making them ideal for this kind of workflow. Because each step is modular, the pipeline can be adapted to support other file types or transformations down the line.

## Step-by-Step: Building the Pipeline

### 1. Uploading Excel Files to Google Cloud Storage
I wrote a Python script to automate the upload of Excel files to a GCS bucket. This script uses the `google-cloud-storage` SDK to authenticate and push files programmatically.

Here's a snippet from the script:

```python
from google.cloud import storage
import os

def upload_excel_files(folder_path, bucket_name, destination_folder):
    client = storage.Client()
    bucket = client.bucket(bucket_name)

    for filename in os.listdir(folder_path):
        if filename.endswith(".xlsx"):
            print(f'Processing {filename}')
            blob = bucket.blob(f"{destination_folder}/{filename}")
            blob.upload_from_filename(os.path.join(folder_path, filename), timeout=600)
            print(f"Uploaded {filename} to {destination_folder}/")
```

This automated step ensures all Excel files in a local directory are uploaded to GCS without manual intervention.

### 2. Reading and Cleaning Excel Files with Cloud Run
Once the files were in GCS, I used a Cloud Run service to clean and process the data. Instead of pulling data directly from GCS inside the Cloud Run function, I used a trigger script that sends a POST request containing the file paths and bucket name.

The Cloud Run function (defined in `main.py`) receives these POST requests, downloads each Excel file as bytes, processes it with pandas, and uploads the cleaned DataFrame to BigQuery. Here's the core logic:

```python
@app.route("/", methods=["POST"])
def clean_and_upload_to_bigquery(request):
    data = request.get_json()
    bucket_name = data["bucket"]
    files = data["files"]  # list of filenames

    storage_client = storage.Client()
    bq_client = bigquery.Client()

    for file_name in files:
        blob = storage_client.bucket(bucket_name).blob(file_name)
        content = blob.download_as_bytes()
        df = pd.read_excel(io.BytesIO(content))

        # Drop "Unnamed" columns
        df = df.loc[:, ~df.columns.str.startswith("Unnamed")]

        # Extract just the file name (e.g., "data1.xlsx")
        base_name = os.path.basename(file_name)

        # Create a sanitized table name
        table_id = f"csv-to-bigquery-demo-457718.tech_jobs.{base_name.replace('.xlsx', '')}"

        job_config = bigquery.LoadJobConfig(write_disposition="WRITE_TRUNCATE")
        job = bq_client.load_table_from_dataframe(df, table_id, job_config=job_config)
        job.result()
        print(f"✅ Loaded {file_name} into {table_id}")

    return jsonify({"status": "success"}), 200
```

### 3. Loading Cleaned Data to BigQuery
As seen above, the BigQuery loading is handled directly in the Cloud Run function using the BigQuery Python client. This enables each cleaned DataFrame to be written to its own table in the `tech_jobs` dataset.

Once in BigQuery, the data is ready for SQL queries or dashboards in tools like Looker Studio.

## Challenges and Lessons Learned
- Excel files can be messy: the dataset I used just had inconsistent headers to deal with, but other Excel files may have merged cells or blank rows requiring careful preprocessing.
- Debugging cloud pipelines is different from local scripts: I learned the importance of logging and isolating errors.

## What I’d Improve Next Time
- Add unit tests to validate data shapes and values before loading to BigQuery.
- Set up logging and alerts for failed uploads or transformation issues.
- Implement a configuration file for more flexible control over input sources and table names.
- Integrate Cloud Scheduler to fully automate execution.

## Results and Why It Matters

This ETL pipeline turns raw Excel files into clean, queryable datasets with minimal manual work. It's ideal for teams that are transitioning from using spreadsheets to using BigQuery or for small teams that rely on spreadsheets but want the power of SQL and cloud analytics.

### Outcomes:
- Automated Excel ingestion to GCS via Python script
- Deployed Cloud Run function to clean and transform data
- Loaded to BigQuery for fast querying and dashboarding

## Try It Yourself

The full code and setup instructions are available on GitHub: [https://github.com/valogonor/ETL-Excel-files-to-BigQuery](https://github.com/valogonor/ETL-Excel-files-to-BigQuery)

## Conclusion

This project demonstrates my ability to design and implement an end-to-end ETL pipeline using cloud-native tools. It highlights not only technical skills in Python and Google Cloud, but also a practical understanding of real-world data challenges. It shows how cloud-native tools can turn spreadsheet chaos into clean, queryable datasets—automatically. If you’re a hiring manager looking for someone who builds scalable data workflows, I’d love to connect.

---

*About the Author: I'm a data enthusiast and engineer with a passion for building robust data systems that bridge the gap between raw inputs and actionable insights. Connect with me on [LinkedIn](https://www.linkedin.com/in/valerieogonor/).*
