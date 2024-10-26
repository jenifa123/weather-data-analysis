# weather-data-analysis

## Overview

This project creates the End to End ETL Pipeline to automate the process of weather data using various AWS Services. The pipeline starts with extracting the data from OpenWeatherMap API, store it in Amazon S3 , transforms the data using AWS Glue and then Loads the data into AWS Redshsift tables from where further analysis can be done. The entire process is automated with the help of AWS CodeBuild CI/CD Pipeline and orchestration is done by Amazon MWAA (Managed Workflows for Apache Airflow).

## Architecture Diagram

 ![architecture](https://github.com/user-attachments/assets/81346878-2df3-495e-9641-9cafe58c54d2)


## AWS Services Used:

1. Amazon S3 :
2. AWS Glue :
3. Amazon Redshift : 
4. AWS CodeBuild :
5. Amazon MWAA : 
