# weather-data-analysis

## Overview

This project creates the End to End ETL Pipeline to automate the process of weather data using various AWS Services. The pipeline starts with extracting the data from OpenWeatherMap API, store it in Amazon S3 , transforms the data using AWS Glue and then Loads the data into AWS Redshsift tables from where further analysis can be done. The entire process is automated with the help of AWS CodeBuild CI/CD Pipeline and orchestration is done by Amazon MWAA (Managed Workflows for Apache Airflow).

## Architecture Diagram

 ![architecture](https://github.com/user-attachments/assets/81346878-2df3-495e-9641-9cafe58c54d2)


## AWS Services Used

1. Amazon S3 : Used to store the weather data extracted from the API in hive style partitioning
2. AWS Glue : Extracts, transforms, and loads (ETL) data from S3, cleaning and preparing it for analysis
3. Amazon Redshift : Serves as a data warehouse to store processed data, making it available for advanced querying and reporting
4. AWS CodeBuild : Automates the build and deployment process for ETL scripts or data processing code, ensuring updates to data pipelines run seamlessly
5. Amazon MWAA : Orchestrates and schedules data workflows, enabling automation of tasks like data ingestion, transformation, and loading into Redshift

## Steps

### Amazon s3

  1. Create one bucket which will store the extracted data from openweathermap API in hive style partitioning.

     ![image](https://github.com/user-attachments/assets/373de84b-4a4f-4078-9677-128b5fc8b3cb)

  2. Create one bucket which will store the dags and requirements.txt which will be required by Amazon MWAA

     ![image](https://github.com/user-attachments/assets/6a55b37d-3330-4116-a163-46ca4887f309)

  3. Create one bucket which is a temporary bucket to store the scripts of glue Job

     ![image](https://github.com/user-attachments/assets/00bb17fd-6125-4f44-974c-585d615ae00b)

### Github Setup

 1. Create a repository in github and clone it.
    
    `git clone 'repository_url'`
    
 2. Create a branch named as 'dev' in your local directory where you have cloned the repository which will be used for development
 
    `git checkout -b dev`
   
 3. Add all the required files in the dev branch and commit to the main branch once done.
 
    ```
    git status
    git add .
    git commit -m 'Commit'
    git push origin dev
    ```
 4. Once all changes are committed to the dev branch, create a pull request from dev branch to main branch and merge the changes.
 
 5. Once the code changes are merged AWS CodeBuild gets triggered.

 ### AWS CodeBuild

  1. Specify the proper github account and the buildspec file which will contain the required dependencies.

     ![image](https://github.com/user-attachments/assets/df251698-85c1-486f-8b49-dc654d901a58)

### Amazon MWAA

  1. Create a environment in airflow with proper configuration. Create the VPC if not created already and attach the airflow cluster to proper subnets.
  2. Run the DAG1 `openweather_api_dag` manually or schedule it and if it is succceded it will trigger the second DAG `transform_redshift_dag`


### Airflow DAGS and its working

**`openweather_api_dag`**
 
 This DAG is used to extract the data from the API and load the data into S3 bucket named as 'weather-data-jn'.

 It is divided into 3 steps:
   - PythonOperator : Used to call the OpenWeatherMap API using pandas library and convert to JSON
   - S3CreateObjectOperator : Used to store the data into 'weather-data-jn' bucket in S3
   - TriggerDagRunOperator : Used to trigger the `transform_redshift_dag` once the data is loaded in S3 using S3CreateObjectOperator

   ![DAG_1](https://github.com/user-attachments/assets/aa67a697-7aef-4a72-955d-0b203c29582b)

**`transform_redshift_dag`**

  This DAG is used to run the Glue JOB that does the transformation and create the redshift table and insert the data into redshift table.

  This DAG has 1 operator:
    - GlueJobOperator : It is used to run the Glue Job named as "glue_transform_task" and inserts the data in Redshift table.

####    DAG2
    
    ![DAG_2](https://github.com/user-attachments/assets/dd4ac0ab-5be9-491e-bbc8-760d9d755a0e)


### Glue Job:

The Glue script `weather-ingestion.py` is used to perform the actual transformation and load the data into Redshift.

Make sure to attach proper IAM Roles with proper permissions attached.

Main  Components:

- **Create DynamicFrame** : Reads the data from S3
- **ApplyMapping**  : Applies the required schema transformation
- **Write DynamicFrame** : To write the data to Redshift after the transformation (`weather_data`)

After the DAG2 `transform_redshift_dag` is completed, check the status of the Glue JOB , if it is a success, the data will be populated in Redshift.

Once the data is loaded, validate the data in the `public.weather_data` table.

#####   Run Status of Glue JOB
    
  ![Glue Job Runs](https://github.com/user-attachments/assets/d136f8bb-82a8-4abc-b3de-2dd164c333d0)
 
#####  Data populated in Redshift DB

  ![Redshift DB](https://github.com/user-attachments/assets/a2096851-dcec-4780-aa7e-779dded577ab)


### Conclusion

This project showcases an end-to-end, scalable solution for ingesting and processing weather data through AWS, combining automation with efficient orchestration and data handling. By utilizing Airflow for managing workflows, Glue for ETL processing, Redshift for centralized data storage, and CodeBuild for continuous integration and delivery, this pipeline achieves streamlined data ingestion and ready-to-use insights for analysis.
