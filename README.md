# DataGamePipe

[![MIT License](https://img.shields.io/badge/License-MIT-green.svg)](https://choosealicense.com/licenses/mit/)

## Autor

- [@jujubalandia](https://www.github.com/jujubalandia)

A End to End Data Engineering Project  for the Data Engineering ZooCamp Course 2025

- [DataGamePipe](#datagamepipe)
- [Backloggd Video Game Analysis Data Pipeline](#backloggd-video-game-analysis-data-pipeline)
  * [Project Overview](#project-overview)
  * [Architecture](#architecture)
  * [Data Pipeline Stages](#data-pipeline-stages)
  * [Data Model](#data-model)
    + [Games source file](#games-source-file)
    + [Developers source file](#developers-source-file)
    + [Platforms source file](#platforms-source-file)
    + [Genres source file](#genres-source-file)
    + [Scores source file](#scores-source-file)
  * [Infrastructure as Code](#infrastructure-as-code)
  * [Error Handling](#error-handling)
  * [Deployment](#deployment)
- [Hands On Time: Backloggd Video Game Analysis Data Pipeline - Implementation Steps](#hands-on-time-backloggd-video-game-analysis-data-pipeline---implementation-steps)
  * [1. Setting up the GCP Development Environment](#1-setting-up-the-gcp-development-environment)
  * [2. Infrastructure Setup with Terraform](#2-infrastructure-setup-with-terraform)
  * [3. Configure Composer and dlt Hub for Data Ingestion](#3-configure-composer-and-dlt-hub-for-data-ingestion)
  * [4. Develop Data Processing & Transformation Scripts](#4-develop-data-processing--transformation-scripts)
  * [5. Automating the Data Pipeline Transformation with Composer](#5-automating-the-data-pipeline-transformation-with-composer)
  * [6. Visualization with Streamlit](#6-visualization-with-streamlit)
  * [Project Summary and Specifications](#project-summary-and-specifications)@Jujubalandia ➜ /workspaces/DataGamePipe (main) $ markdown-toc README.md
- [DataGamePipe](#datagamepipe)
- [Backloggd Video Game Analysis Data Pipeline](#backloggd-video-game-analysis-data-pipeline)
  * [Project Overview](#project-overview)
  * [Architecture](#architecture)
  * [Data Pipeline Stages](#data-pipeline-stages)
  * [Data Model](#data-model)
    + [Games source file](#games-source-file)
    + [Developers source file](#developers-source-file)
    + [Platforms source file](#platforms-source-file)
    + [Genres source file](#genres-source-file)
    + [Scores source file](#scores-source-file)
  * [Infrastructure as Code](#infrastructure-as-code)
  * [Error Handling](#error-handling)
  * [Deployment](#deployment)
- [Hands On Time: Backloggd Video Game Analysis Data Pipeline - Implementation Steps](#hands-on-time-backloggd-video-game-analysis-data-pipeline---implementation-steps)
  * [1. Setting up the GCP Development Environment](#1-setting-up-the-gcp-development-environment)
  * [2. Infrastructure Setup with Terraform](#2-infrastructure-setup-with-terraform)
  * [3. Configure Composer and dlt Hub for Data Ingestion](#3-configure-composer-and-dlt-hub-for-data-ingestion)
  * [4. Develop Data Processing & Transformation Scripts](#4-develop-data-processing--transformation-scripts)
  * [5. Automating the Data Pipeline Transformation with Composer](#5-automating-the-data-pipeline-transformation-with-composer)
  * [6. Visualization with Streamlit](#6-visualization-with-streamlit)
- [Project Summary and Specifications](#project-summary-and-specifications)

# Backloggd Video Game Analysis Data Pipeline

This project builds an end-to-end data pipeline on Google Cloud Platform (GCP) to analyze the Backloggd video game dataset from Kaggle.  The pipeline ingests, processes, transforms, and visualizes data to answer key questions about video game trends and player behavior.

## Project Overview

This project aims to allow video gamers lovers to answer the following questions as example:

* Number of total games released by year
* Categorical data of video game genres released by year
* Top 20 games rated by year
* Top platforms that games were released on by year
* Which game genres, platforms, and developers have the highest total number of players?
* Which game genres, platforms, and developers have the highest average video game ratings?

## Architecture

The pipeline follows an ELT (Extract, Load, Transform) process using the following technologies:

* **Data Source:** Kaggle (Backloggd Dataset)
* **Ingestion:** dlt Hub, Kaggle API
* **Storage (Raw & Silver):** Google Cloud Storage (GCS)
* **Orchestration:** Apache Airflow (Cloud Composer)
* **Data Quality:** SODA framework
* **Data Processing:** Dataflow (Batch, Python) with Soda or Great Expectations for data quality
* **Data Transformation:** dbt Cloud
* **Data Warehousing:** BigQuery (partitioned by year, clustered by platform)
* **Visualization:** Streamlit deployed on Cloud Run
* **Infrastructure as Code:** Terraform

![Architecture](images/BQ-GameZoo-DE.png)


## Data Pipeline Stages

1. **Extract:**
    * dlt Hub extracts raw CSV data from Kaggle using the Kaggle API.
    * Raw data and images are stored in designated GCS buckets.

2. **Load:**
    * Dataflow jobs ingest data from GCS.
    * Data quality checks (using Soda or Great Expectations) are performed within the Dataflow pipeline, skipping records with missing `date` values and including those with future dates.
    * Data is loaded into BigQuery tables, partitioned by `year` and clustered by `platform`.

3. **Transform:**
    * dbt Cloud transforms data within BigQuery, creating staging, intermediate, and mart models.
    * dbt handles data deduplication based on the `id` field.
    * Incremental aggregation logic is implemented in dbt to update yearly aggregates each month.
    * Materialized views are created in BigQuery for optimized query performance and are refreshed monthly.
    * dbt also includes tests and documentation for data quality and pipeline reliability.

4. **Visualize:**
    * A Streamlit application deployed on Cloud Run provides interactive visualizations.
    * Visualizations include line charts, bar charts, and interactive tables with pagination (25 rows per page).
    * Filters allow users to explore data by individual years and single values for genre, platform, and developer/publisher.
    * If selected filters yield no results, the application displays data for all years with a message indicating no data matched the specific filters.
    * The Streamlit application is publicly accessible with no authentication.


## Data Model

The Backloggd dataset is composed by 6 files in csv text format detailed bellow plus a folder with jpg files regards a video game posters

### Games source file

| Field      | Data Type | Description                                            |
|------------|-----------|--------------------------------------------------------|
| id         | INT64     | Video game identifier (primary key)                    |
| name       | STRING    | Name of the video game                                 |
| date       | DATE      | Release date of the video game                         |
| rating     | FLOAT64   | Average rating of the video game                       |
| reviews    | INT64     | Number of reviews                                      |
| plays      | INT64     | Total number of players                                |
| playing    | INT64     | Number of players currently playing                    |
| backlogs   | INT64     | Number of times added to backlog                       |
| wishlists  | INT64     | Number of times added to wishlist                      |
| description | STRING   | Description of the video game                          |

### Developers source file 

| Field      | Data Type | Description                                            |
|------------|-----------|--------------------------------------------------------|
| id         | INT64     | Video game identifier ((foreign key)                   |
| developer  | STRING    | Developer (publisher) of the video game                |

### Platforms source file 

| Field      | Data Type | Description                                            |
|------------|-----------|--------------------------------------------------------|
| id         | INT64     | Video game identifier ((foreign key)                   |
| platform   | STRING    | Gaming platform                                        |

### Genres source file

| Field      | Data Type | Description                                            |
|------------|-----------|--------------------------------------------------------|
| id         | INT64     | Video game identifier ((foreign key)                   |
| genre      | STRING    | Video game genre                                       |

### Scores source file 

| Field      | Data Type | Description                                            |
|------------|-----------|--------------------------------------------------------|
| id         | INT64     | Video game identifier ((foreign key)                   |
| score      | FLOAT64   | User-provided score (0.5 to 5 in increments of 0.5)    |
| amount     | INT64     | Number of users who gave a specific score              |
 

## Infrastructure as Code

Terraform scripts are used to provision and manage all infrastructure components, including:

* GCS Buckets
* BigQuery Datasets
* Cloud Composer
* Cloud Run

[Include links to Terraform scripts or directories here]


## Error Handling

* Dataflow handles missing `date` values by skipping the affected records.
* dbt handles duplicate records using the `id` field.
* The Streamlit application gracefully handles empty filter results by displaying a message and default data.
* Logging and monitoring should be implemented throughout the pipeline to capture and address any other errors.

## Deployment

The Streamlit application is deployed on Cloud Run and is publicly accessible without authentication.

# Hands On Time: Backloggd Video Game Analysis Data Pipeline - Implementation Steps

This section outlines the step-by-step implementation of the Backloggd video game analysis data pipeline on Google Cloud Platform (GCP).

## 1. Setting up the GCP Development Environment

* Create a new Google Cloud Platform (GCP) project.
* Set up a new GitHub Codespaces environment for development.
* Install and configure the Google Cloud SDK (gcloud) and other necessary command-line tools.

## 2. Infrastructure Setup with Terraform

* **Provision Google Cloud Storage (GCS):** Use Terraform to create the necessary GCS buckets for storing raw data and processed files.  Provide Terraform templates with configurable names for each bucket.  
    * folder/filename: `terraform/gcs.tf`
* **Provision BigQuery Datasets:** Create the required BigQuery datasets using Terraform.  Include configuration options for dataset location and access controls.
    * folder/filename: `terraform/bigquery.tf`
* **Provision Google Cloud Composer:** Deploy Cloud Composer (Airflow) using Terraform, specifying the desired configuration (e.g., environment size, location).
    * folder/filename: `terraform/composer.tf`
* **Provision Google Cloud Run:** Set up Cloud Run for deploying the Streamlit application, configuring service settings like scaling and environment variables.
    * folder/filename: `terraform/cloudrun.tf`
  **GCP Variables file:** File to set variables of gcp projec user and roles / permissions used to provisioning all resources bellow
    * folder/filename: `terraform/variables.tf`  

## 3. Configure Composer and dlt Hub for Data Ingestion

* **Configure Airflow Connections:**  In the Cloud Composer Airflow UI, configure connections to:
    * Kaggle API
    * Google Cloud Project
    * dlt Hub
* **Set up Ingestion DAGs:** Create Airflow DAGs using dlt Hub operators to:
    * Download datasets from Kaggle via the Kaggle API.
    * Upload the raw CSV data to the designated GCS bucket.
    * Download game images from the Kaggle API and store them in GCS.

## 4. Develop Data Processing & Transformation Scripts

* **Dataflow Processing:** Develop Dataflow scripts (Python) to:
    * Read raw data from GCS.
    * Perform data cleaning, validation, and preprocessing.  This includes handling missing `date` values (skip records) and incorporating records with future dates.
    * Implement data quality checks using a tool like Soda or Great Expectations.
    * Load processed data into BigQuery tables, partitioned by `year` and clustered by `platform`.
* **dbt Transformation:** Create a dbt Cloud project and develop dbt models to:
    * Build staging, intermediate, and mart models according to the project requirements.
    * Implement data deduplication logic based on the `id` field.
    * Implement incremental aggregation logic for yearly aggregates.
    * Create materialized views for optimized query performance, scheduled for monthly refreshes.
    * Include comprehensive tests and documentation within dbt to ensure data quality and pipeline reliability.

## 5. Automating the Data Pipeline Transformation with Composer

* **Configure dbt Cloud Connection:** In the Airflow UI, configure the connection to dbt Cloud.
* **Orchestrate with Airflow:**  Create DAGs in Composer to automate the data processing and transformation steps:
    * Run the Dataflow pipeline to process and load data into BigQuery.
    * Trigger dbt Cloud to execute the defined models and refresh materialized views.

## 6. Visualization with Streamlit

* **Deploy Streamlit on Cloud Run:**  Set up and deploy the Streamlit application on Cloud Run. Include necessary libraries and dependencies in the Cloud Run deployment configuration.
* **Develop Streamlit Dashboard:** Create an interactive Streamlit dashboard to visualize the processed and transformed data:
    * **Games Dashboard:** Allow users to filter by individual games to view detailed statistics.
    * **Platform Dashboard:**  Enable filtering by game console to explore aggregated statistics.
    * Implement filtering by individual years and single values for genre, platform, and developer/publisher.
    * Include line charts, bar charts, and interactive tables with pagination (25 rows per page).
    * Handle empty filter results by displaying data for all years and a message indicating no data matched the specific filters.

# Project Summary and Specifications

* **Data Source:** Backloggd dataset (Kaggle)
* **Data Types:**  [Refer to the data model section in the main README]
* **Aggregation:** Incremental aggregation with monthly materialized view refreshes
* **Visualization:** Streamlit application with line charts, bar charts, interactive tables, and filtering capabilities
* **Filtering:** By individual years, single genre, platform, and developer/publisher
* **Empty Filter Handling:** Display data for all years with an informative message
* **Deployment:** Streamlit on Cloud Run (public access, no authentication)

## Feedback

If you want to give your two cents don't be shy and let me know [@jujubalandia](https://www.github.com/jujubalandia)

## Contributing

Contributions are welcome!

## License 

[MIT](https://choosealicense.com/licenses/mit/)