# Analyzing Natality Data Using Datalab and BigQuery

In this lab you analyze a large (137 million rows) natality dataset using Google BigQuery and Cloud Datalab. This lab is part of a series of labs on processing scientific data.

1. To launch Datalab, you first create a VM instance and SSH into it. In Cloud Shell, add the following command to create a Datalab:

  datalab create babyweight --zone us-central1-a --no-create-repository
  
2. Open BigQuery Console. In the Google Cloud Console, select Navigation menu > BigQuery. In the Query Editor enter the following query:

  SELECT
  plurality,
  COUNT(1) AS num_babies,
  AVG(weight_pounds) AS ave_weight
FROM
  `bigquery-public-data.samples.natality`
WHERE
  year > 2000 AND year < 2005
GROUP BY
  plurality
  
  
3. Now click Run.

Review the result. How many triplets were born in the US between 2000 and 2005?
  
  
# Visualize data in Cloud Datalab
1. Switch back to your Cloud Shell window. If necessary, wait for Datalab to finish launching. Datalab is ready when you see a message prompting you to use "Web Preview".
Click on the Web Preview icon on the top-left corner of the Cloud Shell ribbon and click on "Change Port".

2. In Datalab, start a new notebook by clicking on the +Notebook icon in the upper left corner.

3. Next, insert the following code to import the BigQuery Python Client Library and initialize a client. The BigQuery client will be used to send and receive messages from the BigQuery API:
  
  !pip install --upgrade google-cloud-bigquery  (updating to the latest version of the BigQuery Python Client Library)

  from google.cloud import bigquery
  client = bigquery.Client()

Run the cell with Shift + Enter.

4. Run a query on the BigQuery natality public dataset, which describes all United States births registered from 1969 to 2008. This query returns the annual count of plural births by plurality (2 for twins, 3 for triplets, etc.).

sql = """
  SELECT
    plurality,
    COUNT(1) AS count,
    year
  FROM
    `bigquery-public-data.samples.natality`
  WHERE
    NOT IS_NAN(plurality) AND plurality > 1
  GROUP BY
    plurality, year
  ORDER BY
    count DESC
"""
df = client.query(sql).to_dataframe()
df.head()

Run the cell with Shift + Enter.

The head of the DataFrame (the first 5 rows) is displayed below the code cell. Full results are available for further analysis in a Pandas DataFrame.

5. Insert the following code into the next cell to pivot the data and create a stacked bar chart of the count of plural births over time:

pivot_table = df.pivot(index='year', columns='plurality', values='count')
pivot_table.plot(kind='bar', stacked=True, figsize=(15,7));

6. Next, take a look at baby weight by gender. In the next cell, enter the following, then run the cell:

sql = """
  SELECT
    is_male,
    AVG(weight_pounds) AS ave_weight
  FROM
    `bigquery-public-data.samples.natality`
  GROUP BY
    is_male
"""
df = client.query(sql).to_dataframe()
df.plot(x='is_male', y='ave_weight', kind='bar');

--------------------------------------------------------------------------------------------------------------

Are male babies heavier or lighter than female babies? Does this align with your expectations?

For your last visualization, see how the baby's weight fluctuates according to the number of gestation weeks.
Enter the following into the next cell and run it:

sql = """
  SELECT
    gestation_weeks,
    AVG(weight_pounds) AS ave_weight
  FROM
    `bigquery-public-data.samples.natality`
  WHERE
    NOT IS_NAN(gestation_weeks) AND gestation_weeks <> 99
  GROUP BY
    gestation_weeks
  ORDER BY
    gestation_weeks
"""
df = client.query(sql).to_dataframe()
df.plot(x='gestation_weeks', y='ave_weight', kind='bar');


Note: Because the gestation_weeks field allows null values and stores unknown values as 99, this query excludes records where gestation_weeks is null or 99.

Now you have a chart that shows how the weight of the baby relates to the number of weeks of gestation.
