#  E-Commerce Sales

## Project Overview

**Project Title**:  E-Commerce Sales 
**Database**: [[E-Commerce Sales]](https://www.kaggle.com/datasets/thedevastator/unlock-profits-with-e-commerce-sales-data/data)
**Data capacity**: Includes 5 files, total 178,343 thousand rows of data

This project aims to analyze Amazon's e-commerce sales in 2022. The data is collected on the Kaggle website in the form of raw data. Through processing and analysis, some results are drawn as follows:

## Result

1. **Monthly Revenue**: The month with the highest revenue was June with revenue reaching 13,776,846,954 INR
2. **Best selling products**: The product with the highest number of orders has SKU code SET268-KR-NP-L size L, color OFF WHITE with 5525, sales reached 7,714,850 INR
3. **Area with most orders**: Top 10 cities with most orders and highest revenue are bengaluru, hyderabad, mumbai, new delhi, chennai, pune, kolkata, gurugram, thane, noida


## Project Structure

### 1. Prepare data

Raw data collected from Kaggle website

### 2. Data Exploration & Cleaning

Here, I will import the collected raw data into a separate data warehouse that I created earlier. Here, I realized that the data mainly encountered the following problems:
- Duplicate data rows
- Data errors
- Records in some data rows are incomplete
- Records in the same column are not written in the same way
- Data types in columns are not consistent
Then, I performed the commands `Alter table`, `Update`, `Cast`, `Delete`,... to solve the above problems

### 3. Data Analysis & Findings

To analyze, I use SQL server tool to query data Answer Business Questions. The query statements are mentioned in the `E-Commerce Sales file`.

Thanks for taking the time to read my personal project!
