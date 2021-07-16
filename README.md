# Amazon_Vine_Analysis
Columbia Data Science Module 16

## Overview
For this project, I was instructed to conduct an analysis of product reviews from the Amazon Vine program - a program in which participants are paid to write reviews. In order to complete this project, I created an RDS on AWS. Then, using PySpark in Google Colab, I pushed the Amazon product review data to my local PostgreSQL database. Finally, I exported the dataframe as a .csv, read it into Jupyter Notebook, and used Pandas to conduct an analysis of the data. 

The question I answer in this project is: do paid reviewers give 5-star reviews at a higher rate than unpaid reviewers? 

### Resources
Software/tools: PGAdmin 4, SQL, AWS, Jupyter Notebook, Google Colab

Python packages: PySpark, pandas, numpy, spark 3.0

Data: The pet products Amazon review dataset, available [here](https://s3.amazonaws.com/amazon-reviews-pds/tsv/index.txt).

## Analysis 

### Perform ETL on Amazon Product Reviews

I began my analysis by creating a database with Amazon RDS. Next, I used the SQL query below to create tables for my database. 

![sql_create_tables.png](Resources/sql_create_tables.PNG)

In Google Colab, I initialized the data transfer by importing the necessary packages and Postgres drivers. 

![initialize.png](Resources/initialize.PNG)

Next, I loaded the AWS data into the Spark data frame. 

![aws_load.png](Resources/aws_load.PNG)

Then, a table was made corresponding to each table in Postgres. 

![df_to_match_tables.png](Resources/df_to_match_tables.PNG)

This process was repeated for the products_table and vine_table. The same process was used for review_id_table, but the date was also changed to datetime format. 

```
review_id_df = df.select(["review_id", "customer_id", "product_id", "product_parent", to_date("review_date", 'yyyy-MM-dd').alias("review_date")])
```
I connected to AWS RDS and wrote each data frame to its table. (Note: my password is not actually "password" - that's a stand-in.  I wanted to let people know lest my cyber security integrity never be trusted again.)

![write_to_aws.png](Resources/write_to_aws.PNG)

## Results
* **Vine reviews vs. non-Vine reviews**

I filtered the paid review dataset with the code below (I had previously cleaned the data set):

```
vine_df_clean_filtered_total_votes_ratio_unpaid = vine_df_clean_filtered_total_votes_ratio[vine_df_clean_filtered_total_votes_ratio['vine'] == False]
```
This yielded 170 paid Vine reviews for pet products. 

![total_paid.png](Resources/total_paid.PNG)

I ran another filter to create a data set with just unpaid reviews for pet products.
```
vine_df_clean_filtered_total_votes_ratio_unpaid = vine_df_clean_filtered_total_votes_ratio[vine_df_clean_filtered_total_votes_ratio['vine'] == False]
```

There are 37,840 unpaid reviews for pet products.

![total_unpaid.png](Resources/total_unpaid.PNG)

* **5-Star reviews in Vine vs. non-Vine**

I created a new dataframe that contained only 5-star paid Vine reviews. 
```
total_paid_5_star_df = vine_df_clean_filtered_total_votes_ratio_paid[vine_df_clean_filtered_total_votes_ratio_paid['star_rating'] == 5]
```

![paid_5_star.png](Resources/paid_5_star.PNG)

This left 65 out of 170 total paid reviews as 5-star. 

Next, I created a new dataframe containing only 5-star unpaid reviews.
```
total_unpaid_5_star_df = vine_df_clean_filtered_total_votes_ratio_unpaid[vine_df_clean_filtered_total_votes_ratio_unpaid['star_rating'] == 5]
```

![unpaid_5_star.png](Resources/unpaid_5_star.PNG)

* **Percentage of 5-star ratings in paid vs. unpaid reviews**

Finally, I calculated the percentage of paid reviews that were 5-star. 

![percent_5_star_paid.png](Resources/percent_5_star_paid.PNG)

Approximately 38.2% of paid reviews were 5-star. 

Then, I calculated the percentage of unpaid reviews that were 5-star.

![percent_5_star_unpaid.png](Resources/percent_5_star_unpaid.PNG)

Roughly 54.5% of unpaid reviews were 5-star. 

## Summary

There is no evidence to suggest that paid Vine reviewers are biased positively toward pet products. In fact, they are significantly less positive toward products than their unpaid counterparts: just 38.2% of their reviews were 5-star, compared with 54.5% of the unpaid reviews. 

In order to confirm this, it would be helpful to do further analysis on the full dataset. Initially, the dataset contained well over 2.6 million reviews. However, I significantly winnowed this dataset down before doing my analysis. After filtering out reviews that didn't receive at least 20 votes and filtering out reviews that didn't have a positive vote ratio of at least 50%, only approximately 38,000 reviews remained. It's possible that this created selection bias: maybe users do not find 5-star reviews from paid reviewers trustworthy, so they do not vote for them. We would need to do analysis on the larger data set to make sure this analysis is not just a fluke of a highly curated dataset. 
