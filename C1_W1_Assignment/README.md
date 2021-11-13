# Register and visualize dataset

### Introduction

In this lab you will ingest and transform the customer product reviews dataset. Then you will use AWS data stack services such as AWS Glue and Amazon Athena for ingesting and querying the dataset. Finally you will use AWS Data Wrangler to analyze the dataset and plot some visuals extracting insights.

### Table of Contents

- [1. Ingest and transform the public dataset](#c1w1-1.)
  - [1.1. List the dataset files in the public S3 bucket](#c1w1-1.1.)
    - [Exercise 1](#c1w1-ex-1)
  - [1.2. Copy the data locally to the notebook](#c1w1-1.2.)
  - [1.3. Transform the data](#c1w1-1.3.)
  - [1.4 Write the data to a CSV file](#c1w1-1.4.)
- [2. Register the public dataset for querying and visualizing](#c1w1-2.)
  - [2.1. Register S3 dataset files as a table for querying](#c1w1-2.1.)
    - [Exercise 2](#c1w1-ex-2)
  - [2.2. Create default S3 bucket for Amazon Athena](#c1w1-2.2.)
- [3. Visualize data](#c1w1-3.)
  - [3.1. Preparation for data visualization](#c1w1-3.1.)
  - [3.2. How many reviews per sentiment?](#c1w1-3.2.)
    - [Exercise 3](#c1w1-ex-3)
  - [3.3. Which product categories are highest rated by average sentiment?](#c1w1-3.3.)
  - [3.4. Which product categories have the most reviews?](#c1w1-3.4.)
    - [Exercise 4](#c1w1-ex-4)
  - [3.5. What is the breakdown of sentiments per product category?](#c1w1-3.5.)
  - [3.6. Analyze the distribution of review word counts](#c1w1-3.6.)

Let's install the required modules first.


```python
# please ignore warning messages during the installation
!pip install --disable-pip-version-check -q sagemaker==2.35.0
!pip install --disable-pip-version-check -q pandas==1.1.4
!pip install --disable-pip-version-check -q awswrangler==2.7.0
!pip install --disable-pip-version-check -q numpy==1.18.5
!pip install --disable-pip-version-check -q seaborn==0.11.0
!pip install --disable-pip-version-check -q matplotlib===3.3.3
```

    /opt/conda/lib/python3.7/site-packages/secretstorage/dhcrypto.py:16: CryptographyDeprecationWarning: int_from_bytes is deprecated, use int.from_bytes instead
      from cryptography.utils import int_from_bytes
    /opt/conda/lib/python3.7/site-packages/secretstorage/util.py:25: CryptographyDeprecationWarning: int_from_bytes is deprecated, use int.from_bytes instead
      from cryptography.utils import int_from_bytes
    [33mWARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv[0m
    /opt/conda/lib/python3.7/site-packages/secretstorage/dhcrypto.py:16: CryptographyDeprecationWarning: int_from_bytes is deprecated, use int.from_bytes instead
      from cryptography.utils import int_from_bytes
    /opt/conda/lib/python3.7/site-packages/secretstorage/util.py:25: CryptographyDeprecationWarning: int_from_bytes is deprecated, use int.from_bytes instead
      from cryptography.utils import int_from_bytes
    [33mWARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv[0m
    /opt/conda/lib/python3.7/site-packages/secretstorage/dhcrypto.py:16: CryptographyDeprecationWarning: int_from_bytes is deprecated, use int.from_bytes instead
      from cryptography.utils import int_from_bytes
    /opt/conda/lib/python3.7/site-packages/secretstorage/util.py:25: CryptographyDeprecationWarning: int_from_bytes is deprecated, use int.from_bytes instead
      from cryptography.utils import int_from_bytes
    [33mWARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv[0m
    /opt/conda/lib/python3.7/site-packages/secretstorage/dhcrypto.py:16: CryptographyDeprecationWarning: int_from_bytes is deprecated, use int.from_bytes instead
      from cryptography.utils import int_from_bytes
    /opt/conda/lib/python3.7/site-packages/secretstorage/util.py:25: CryptographyDeprecationWarning: int_from_bytes is deprecated, use int.from_bytes instead
      from cryptography.utils import int_from_bytes
    [33mWARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv[0m
    /opt/conda/lib/python3.7/site-packages/secretstorage/dhcrypto.py:16: CryptographyDeprecationWarning: int_from_bytes is deprecated, use int.from_bytes instead
      from cryptography.utils import int_from_bytes
    /opt/conda/lib/python3.7/site-packages/secretstorage/util.py:25: CryptographyDeprecationWarning: int_from_bytes is deprecated, use int.from_bytes instead
      from cryptography.utils import int_from_bytes
    [33mWARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv[0m
    /opt/conda/lib/python3.7/site-packages/secretstorage/dhcrypto.py:16: CryptographyDeprecationWarning: int_from_bytes is deprecated, use int.from_bytes instead
      from cryptography.utils import int_from_bytes
    /opt/conda/lib/python3.7/site-packages/secretstorage/util.py:25: CryptographyDeprecationWarning: int_from_bytes is deprecated, use int.from_bytes instead
      from cryptography.utils import int_from_bytes
    [33mWARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv[0m


<a name='c1w1-1.'></a>
# 1. Ingest and transform the public dataset

The dataset [Women's Clothing Reviews](https://www.kaggle.com/nicapotato/womens-ecommerce-clothing-reviews) has been chosen as the main dataset.

It is shared in a public Amazon S3 bucket, and is available as a comma-separated value (CSV) text format:

`s3://dlai-practical-data-science/data/raw/womens_clothing_ecommerce_reviews.csv`

<a name='c1w1-1.1.'></a>
### 1.1. List the dataset files in the public S3 bucket

The [AWS Command Line Interface (CLI)](https://awscli.amazonaws.com/v2/documentation/api/latest/index.html) is a unified tool to manage your AWS services. With just one tool, you can control multiple AWS services from the command line and automate them through scripts. You will use it to list the dataset files.

**View dataset files in CSV format**

```aws s3 ls [bucket_name]``` function lists all objects in the S3 bucket. Let's use it to view the reviews data files in CSV format:

<a name='c1w1-ex-1'></a>
### Exercise 1

View the list of the files available in the public bucket `s3://dlai-practical-data-science/data/raw/`.

**Instructions**:
Use `aws s3 ls [bucket_name]` function. To run the AWS CLI command from the notebook you will need to put an exclamation mark in front of it: `!aws`. You should see the data file `womens_clothing_ecommerce_reviews.csv` in the list.


```python
### BEGIN SOLUTION - DO NOT delete this comment for grading purposes
! aws s3 ls dlai-practical-data-science/data/raw/
### END SOLUTION - DO NOT delete this comment for grading purposes

# EXPECTED OUTPUT
# ... womens_clothing_ecommerce_reviews.csv
```

    2021-04-30 02:21:06    8457214 womens_clothing_ecommerce_reviews.csv


<a name='c1w1-1.2.'></a>
### 1.2. Copy the data locally to the notebook

```aws s3 cp [bucket_name/file_name] [file_name]``` function copies the file from the S3 bucket into the local environment or into another S3 bucket. Let's use it to copy the file with the dataset locally.


```python
!aws s3 cp s3://dlai-practical-data-science/data/raw/womens_clothing_ecommerce_reviews.csv ./womens_clothing_ecommerce_reviews.csv
```

    download: s3://dlai-practical-data-science/data/raw/womens_clothing_ecommerce_reviews.csv to ./womens_clothing_ecommerce_reviews.csv


Now use the Pandas dataframe to load and preview the data.


```python
import pandas as pd
import csv

df = pd.read_csv('./womens_clothing_ecommerce_reviews.csv',
                 index_col=0)

df.shape
```




    (23486, 10)




```python
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Clothing ID</th>
      <th>Age</th>
      <th>Title</th>
      <th>Review Text</th>
      <th>Rating</th>
      <th>Recommended IND</th>
      <th>Positive Feedback Count</th>
      <th>Division Name</th>
      <th>Department Name</th>
      <th>Class Name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>847</td>
      <td>33</td>
      <td>Cute, crisp shirt</td>
      <td>If this product was in petite  i would get the...</td>
      <td>4</td>
      <td>1</td>
      <td>2</td>
      <td>General</td>
      <td>Tops</td>
      <td>Blouses</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1080</td>
      <td>34</td>
      <td>NaN</td>
      <td>Love this dress!  it's sooo pretty.  i happene...</td>
      <td>5</td>
      <td>1</td>
      <td>4</td>
      <td>General</td>
      <td>Dresses</td>
      <td>Dresses</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1077</td>
      <td>60</td>
      <td>Some major design flaws</td>
      <td>I had such high hopes for this dress and reall...</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>General</td>
      <td>Dresses</td>
      <td>Dresses</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1049</td>
      <td>50</td>
      <td>My favorite buy!</td>
      <td>I love  love  love this jumpsuit. it's fun  fl...</td>
      <td>5</td>
      <td>1</td>
      <td>0</td>
      <td>General Petite</td>
      <td>Bottoms</td>
      <td>Pants</td>
    </tr>
    <tr>
      <th>4</th>
      <td>847</td>
      <td>47</td>
      <td>Flattering shirt</td>
      <td>This shirt is very flattering to all due to th...</td>
      <td>5</td>
      <td>1</td>
      <td>6</td>
      <td>General</td>
      <td>Tops</td>
      <td>Blouses</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>23481</th>
      <td>1104</td>
      <td>34</td>
      <td>Great dress for many occasions</td>
      <td>I was very happy to snag this dress at such a ...</td>
      <td>5</td>
      <td>1</td>
      <td>0</td>
      <td>General Petite</td>
      <td>Dresses</td>
      <td>Dresses</td>
    </tr>
    <tr>
      <th>23482</th>
      <td>862</td>
      <td>48</td>
      <td>Wish it was made of cotton</td>
      <td>It reminds me of maternity clothes. soft  stre...</td>
      <td>3</td>
      <td>1</td>
      <td>0</td>
      <td>General Petite</td>
      <td>Tops</td>
      <td>Knits</td>
    </tr>
    <tr>
      <th>23483</th>
      <td>1104</td>
      <td>31</td>
      <td>Cute, but see through</td>
      <td>This fit well  but the top was very see throug...</td>
      <td>3</td>
      <td>0</td>
      <td>1</td>
      <td>General Petite</td>
      <td>Dresses</td>
      <td>Dresses</td>
    </tr>
    <tr>
      <th>23484</th>
      <td>1084</td>
      <td>28</td>
      <td>Very cute dress, perfect for summer parties an...</td>
      <td>I bought this dress for a wedding i have this ...</td>
      <td>3</td>
      <td>1</td>
      <td>2</td>
      <td>General</td>
      <td>Dresses</td>
      <td>Dresses</td>
    </tr>
    <tr>
      <th>23485</th>
      <td>1104</td>
      <td>52</td>
      <td>Please make more like this one!</td>
      <td>This dress in a lovely platinum is feminine an...</td>
      <td>5</td>
      <td>1</td>
      <td>22</td>
      <td>General Petite</td>
      <td>Dresses</td>
      <td>Dresses</td>
    </tr>
  </tbody>
</table>
<p>23486 rows × 10 columns</p>
</div>



<a name='c1w1-1.3.'></a>
### 1.3. Transform the data
To simplify the task, you will transform the data into a comma-separated value (CSV) file that contains only a `review_body`, `product_category`, and `sentiment` derived from the original data.


```python
df_transformed = df.rename(columns={'Review Text': 'review_body',
                                    'Rating': 'star_rating',
                                    'Class Name': 'product_category'})
df_transformed.drop(columns=['Clothing ID', 'Age', 'Title', 'Recommended IND', 'Positive Feedback Count', 'Division Name', 'Department Name'],
                    inplace=True)

df_transformed.dropna(inplace=True)

df_transformed.shape
```




    (22628, 3)



Now convert the `star_rating` into the `sentiment` (positive, neutral, negative), which later on will be for the prediction.


```python
def to_sentiment(star_rating):
    if star_rating in {1, 2}: # negative
        return -1 
    if star_rating == 3:      # neutral
        return 0
    if star_rating in {4, 5}: # positive
        return 1

# transform star_rating into the sentiment
df_transformed['sentiment'] = df_transformed['star_rating'].apply(lambda star_rating: 
    to_sentiment(star_rating=star_rating) 
)

# drop the star rating column
df_transformed.drop(columns=['star_rating'],
                    inplace=True)

# remove reviews for product_categories with < 10 reviews
df_transformed = df_transformed.groupby('product_category').filter(lambda reviews : len(reviews) > 10)[['sentiment', 'review_body', 'product_category']]

df_transformed.shape
```




    (22626, 3)




```python
# preview the results
df_transformed
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>sentiment</th>
      <th>review_body</th>
      <th>product_category</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>If this product was in petite  i would get the...</td>
      <td>Blouses</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>Love this dress!  it's sooo pretty.  i happene...</td>
      <td>Dresses</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>I had such high hopes for this dress and reall...</td>
      <td>Dresses</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>I love  love  love this jumpsuit. it's fun  fl...</td>
      <td>Pants</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1</td>
      <td>This shirt is very flattering to all due to th...</td>
      <td>Blouses</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>23481</th>
      <td>1</td>
      <td>I was very happy to snag this dress at such a ...</td>
      <td>Dresses</td>
    </tr>
    <tr>
      <th>23482</th>
      <td>0</td>
      <td>It reminds me of maternity clothes. soft  stre...</td>
      <td>Knits</td>
    </tr>
    <tr>
      <th>23483</th>
      <td>0</td>
      <td>This fit well  but the top was very see throug...</td>
      <td>Dresses</td>
    </tr>
    <tr>
      <th>23484</th>
      <td>0</td>
      <td>I bought this dress for a wedding i have this ...</td>
      <td>Dresses</td>
    </tr>
    <tr>
      <th>23485</th>
      <td>1</td>
      <td>This dress in a lovely platinum is feminine an...</td>
      <td>Dresses</td>
    </tr>
  </tbody>
</table>
<p>22626 rows × 3 columns</p>
</div>



<a name='c1w1-1.4.'></a>
### 1.4 Write the data to a CSV file


```python
df_transformed.to_csv('./womens_clothing_ecommerce_reviews_transformed.csv', 
                      index=False)
```


```python
!head -n 5 ./womens_clothing_ecommerce_reviews_transformed.csv
```

    sentiment,review_body,product_category
    1,If this product was in petite  i would get the petite. the regular is a little long on me but a tailor can do a simple fix on that.     fits nicely! i'm 5'4  130lb and pregnant so i bough t medium to grow into.     the tie can be front or back so provides for some nice flexibility on form fitting.,Blouses
    1,"Love this dress!  it's sooo pretty.  i happened to find it in a store  and i'm glad i did bc i never would have ordered it online bc it's petite.  i bought a petite and am 5'8"".  i love the length on me- hits just a little below the knee.  would definitely be a true midi on someone who is truly petite.",Dresses
    0,I had such high hopes for this dress and really wanted it to work for me. i initially ordered the petite small (my usual size) but i found this to be outrageously small. so small in fact that i could not zip it up! i reordered it in petite medium  which was just ok. overall  the top half was comfortable and fit nicely  but the bottom half had a very tight under layer and several somewhat cheap (net) over layers. imo  a major design flaw was the net over layer sewn directly into the zipper - it c,Dresses
    1,I love  love  love this jumpsuit. it's fun  flirty  and fabulous! every time i wear it  i get nothing but great compliments!,Pants


<a name='c1w1-2.'></a>
# 2. Register the public dataset for querying and visualizing
You will register the public dataset into an S3-backed database table so you can query and visualize our dataset at scale. 

<a name='c1w1-2.1.'></a>
### 2.1. Register S3 dataset files as a table for querying
Let's import required modules.

`boto3` is the AWS SDK for Python to create, configure, and manage AWS services, such as Amazon Elastic Compute Cloud (Amazon EC2) and Amazon Simple Storage Service (Amazon S3). The SDK provides an object-oriented API as well as low-level access to AWS services. 

`sagemaker` is the SageMaker Python SDK which provides several high-level abstractions for working with the Amazon SageMaker.


```python
import boto3
import sagemaker
import pandas as pd
import numpy as np
import botocore

config = botocore.config.Config(user_agent_extra='dlai-pds/c1/w1')

# low-level service client of the boto3 session
sm = boto3.client(service_name='sagemaker', 
                  config=config)

sess = sagemaker.Session(sagemaker_client=sm)                         

bucket = sess.default_bucket()
role = sagemaker.get_execution_role()
region = sess.boto_region_name
account_id = sess.account_id

print('S3 Bucket: {}'.format(bucket))
print('Region: {}'.format(region))
print('Account ID: {}'.format(account_id))
```

    S3 Bucket: sagemaker-us-east-1-139279355515
    Region: us-east-1
    Account ID: <bound method Session.account_id of <sagemaker.session.Session object at 0x7faf4a612bd0>>


Review the empty bucket which was created automatically for this account.

**Instructions**: 
- open the link
- click on the S3 bucket name `sagemaker-us-east-1-ACCOUNT`
- check that it is empty at this stage


```python
from IPython.core.display import display, HTML

display(HTML('<b>Review <a target="top" href="https://s3.console.aws.amazon.com/s3/home?region={}#">Amazon S3 buckets</a></b>'.format(region)))
```


<b>Review <a target="top" href="https://s3.console.aws.amazon.com/s3/home?region=us-east-1#">Amazon S3 buckets</a></b>


Copy the file into the S3 bucket.


```python
!aws s3 cp ./womens_clothing_ecommerce_reviews_transformed.csv s3://$bucket/data/transformed/womens_clothing_ecommerce_reviews_transformed.csv
```

    upload: ./womens_clothing_ecommerce_reviews_transformed.csv to s3://sagemaker-us-east-1-139279355515/data/transformed/womens_clothing_ecommerce_reviews_transformed.csv


Review the bucket with the file we uploaded above.

**Instructions**: 
- open the link
- check that the CSV file is located in the S3 bucket
- check the location directory structure is the same as in the CLI command above
- click on the file name and see the available information about the file (region, size, S3 URI, Amazon Resource Name (ARN))


```python
from IPython.core.display import display, HTML

display(HTML('<b>Review <a target="top" href="https://s3.console.aws.amazon.com/s3/buckets/{}?region={}&prefix=data/transformed/#">Amazon S3 buckets</a></b>'.format(bucket, region)))
```


<b>Review <a target="top" href="https://s3.console.aws.amazon.com/s3/buckets/sagemaker-us-east-1-139279355515?region=us-east-1&prefix=data/transformed/#">Amazon S3 buckets</a></b>


**Import AWS Data Wrangler**

[AWS Data Wrangler](https://github.com/awslabs/aws-data-wrangler) is an AWS Professional Service open source python initiative that extends the power of Pandas library to AWS connecting dataframes and AWS data related services (Amazon Redshift, AWS Glue, Amazon Athena, Amazon EMR, Amazon QuickSight, etc).

Built on top of other open-source projects like Pandas, Apache Arrow, Boto3, SQLAlchemy, Psycopg2 and PyMySQL, it offers abstracted functions to execute usual ETL tasks like load/unload data from data lakes, data warehouses and databases.

Review the AWS Data Wrangler documentation: https://aws-data-wrangler.readthedocs.io/en/stable/


```python
import awswrangler as wr
```

**Create AWS Glue Catalog database**

The data catalog features of **AWS Glue** and the inbuilt integration to Amazon S3 simplify the process of identifying data and deriving the schema definition out of the discovered data. Using AWS Glue crawlers within your data catalog, you can traverse your data stored in Amazon S3 and build out the metadata tables that are defined in your data catalog.

Here you will use `wr.catalog.create_database` function to create a database with the name `dsoaws_deep_learning` ("dsoaws" stands for "Data Science on AWS").


```python
wr.catalog.create_database(
    name='dsoaws_deep_learning',
    exist_ok=True
)
```


```python
dbs = wr.catalog.get_databases()

for db in dbs:
    print("Database name: " + db['Name'])
```

    Database name: dsoaws_deep_learning


Review the created database in the AWS Glue Catalog.

**Instructions**:
- open the link
- on the left side panel notice that you are in the AWS Glue -> Data Catalog -> Databases
- check that the database `dsoaws_deep_learning` has been created
- click on the name of the database
- click on the `Tables in dsoaws_deep_learning` link to see that there are no tables


```python
from IPython.core.display import display, HTML

display(HTML('<b>Review <a target="top" href="https://console.aws.amazon.com/glue/home?region={}#catalog:tab=databases">AWS Glue Databases</a></b>'.format(region)))
```


<b>Review <a target="top" href="https://console.aws.amazon.com/glue/home?region=us-east-1#catalog:tab=databases">AWS Glue Databases</a></b>


**Register CSV data with AWS Glue Catalog**

<a name='c1w1-ex-2'></a>
### Exercise 2

Register CSV data with AWS Glue Catalog.

**Instructions**:
Use ```wr.catalog.create_csv_table``` function with the following parameters
```python
res = wr.catalog.create_csv_table(
    database='...', # AWS Glue Catalog database name
    path='s3://{}/data/transformed/'.format(bucket), # S3 object path for the data
    table='reviews', # registered table name
    columns_types={
        'sentiment': 'int',        
        'review_body': 'string',
        'product_category': 'string'      
    },
    mode='overwrite',
    skip_header_line_count=1,
    sep=','    
)
```


```python
bucket
```




    'sagemaker-us-east-1-139279355515'




```python
wr.catalog.create_csv_table(
    ### BEGIN SOLUTION - DO NOT delete this comment for grading purposes
    database='dsoaws_deep_learning', # Replace None
    ### END SOLUTION - DO NOT delete this comment for grading purposes
    path='s3://{}/data/transformed/'.format(bucket), 
    table="reviews",    
    columns_types={
        'sentiment': 'int',        
        'review_body': 'string',
        'product_category': 'string'      
    },
    mode='overwrite',
    skip_header_line_count=1,
    sep=','
)
```

Review the registered table in the AWS Glue Catalog.

**Instructions**:
- open the link
- on the left side panel notice that you are in the AWS Glue -> Data Catalog -> Databases -> Tables
- check that you can see the table `reviews` from the database `dsoaws_deep_learning` in the list
- click on the name of the table
- explore the available information about the table (name, database, classification, location, schema etc.)


```python
from IPython.core.display import display, HTML

display(HTML('<b>Review <a target="top" href="https://console.aws.amazon.com/glue/home?region={}#">AWS Glue Catalog</a></b>'.format(region)))
```


<b>Review <a target="top" href="https://console.aws.amazon.com/glue/home?region=us-east-1#">AWS Glue Catalog</a></b>


Review the table shape:


```python
table = wr.catalog.table(database='dsoaws_deep_learning',
                         table='reviews')
table
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Column Name</th>
      <th>Type</th>
      <th>Partition</th>
      <th>Comment</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>sentiment</td>
      <td>int</td>
      <td>False</td>
      <td></td>
    </tr>
    <tr>
      <th>1</th>
      <td>review_body</td>
      <td>string</td>
      <td>False</td>
      <td></td>
    </tr>
    <tr>
      <th>2</th>
      <td>product_category</td>
      <td>string</td>
      <td>False</td>
      <td></td>
    </tr>
  </tbody>
</table>
</div>



<a name='c1w1-2.2.'></a>
### 2.2. Create default S3 bucket for Amazon Athena

Amazon Athena requires this S3 bucket to store temporary query results and improve performance of subsequent queries.

The contents of this bucket are mostly binary and human-unreadable. 


```python
# S3 bucket name
wr.athena.create_athena_bucket()

# EXPECTED OUTPUT
# 's3://aws-athena-query-results-ACCOUNT-REGION/'
```




    's3://aws-athena-query-results-139279355515-us-east-1/'



<a name='c1w1-3.'></a>
# 3. Visualize data

**Reviews dataset - column descriptions**

- `sentiment`: The review's sentiment (-1, 0, 1).
- `product_category`: Broad product category that can be used to group reviews (in this case digital videos).
- `review_body`: The text of the review.

<a name='c1w1-3.1.'></a>
### 3.1. Preparation for data visualization

**Imports**


```python
import numpy as np
import seaborn as sns

import matplotlib.pyplot as plt
%matplotlib inline
%config InlineBackend.figure_format='retina'
```

**Settings**

Set AWS Glue database and table name.


```python
# Do not change the database and table names - they are used for grading purposes!
database_name = 'dsoaws_deep_learning'
table_name = 'reviews'
```

Set seaborn parameters. You can review seaborn documentation following the [link](https://seaborn.pydata.org/index.html).


```python
sns.set_style = 'seaborn-whitegrid'

sns.set(rc={"font.style":"normal",
            "axes.facecolor":"white",
            'grid.color': '.8',
            'grid.linestyle': '-',
            "figure.facecolor":"white",
            "figure.titlesize":20,
            "text.color":"black",
            "xtick.color":"black",
            "ytick.color":"black",
            "axes.labelcolor":"black",
            "axes.grid":True,
            'axes.labelsize':10,
            'xtick.labelsize':10,
            'font.size':10,
            'ytick.labelsize':10})
```

Helper code to display values on barplots:

**Run SQL queries using Amazon Athena**

**Amazon Athena** lets you query data in Amazon S3 using a standard SQL interface. It reflects the databases and tables in the AWS Glue Catalog. You can create interactive queries and perform any data manipulations required for further downstream processing.

Standard SQL query can be saved as a string and then passed as a parameter into the Athena query. Run the following cells as an example to count the total number of reviews by sentiment. The SQL query here will take the following form:

```sql
SELECT column_name, COUNT(column_name) as new_column_name
FROM table_name
GROUP BY column_name
ORDER BY column_name
```

If you are not familiar with the SQL query statements, you can review some tutorials following the [link](https://www.w3schools.com/sql/default.asp).

<a name='c1w1-3.2.'></a>
### 3.2. How many reviews per sentiment?

Set the SQL statement to find the count of sentiments:


```python
statement_count_by_sentiment = """
SELECT sentiment, COUNT(sentiment) AS count_sentiment
FROM reviews
GROUP BY sentiment
ORDER BY sentiment
"""

print(statement_count_by_sentiment)
```

    
    SELECT sentiment, COUNT(sentiment) AS count_sentiment
    FROM reviews
    GROUP BY sentiment
    ORDER BY sentiment
    


Query data in Amazon Athena database cluster using the prepared SQL statement:


```python
df_count_by_sentiment = wr.athena.read_sql_query(
    sql=statement_count_by_sentiment,
    database=database_name
)

print(df_count_by_sentiment)
```

       sentiment  count_sentiment
    0         -1             2370
    1          0             2823
    2          1            17433


Preview the results of the query:


```python
df_count_by_sentiment.plot(kind='bar', x='sentiment', y='count_sentiment', rot=0)
```




    <AxesSubplot:xlabel='sentiment'>




![png](output_70_1.png)


<a name='c1w1-ex-3'></a>
### Exercise 3

Use Amazon Athena query with the standard SQL statement passed as a parameter, to calculate the total number of reviews per `product_category` in the table ```reviews```.

**Instructions**: Pass the SQL statement of the form

```sql
SELECT category_column, COUNT(column_name) AS new_column_name
FROM table_name
GROUP BY category_column
ORDER BY new_column_name DESC
```

as a triple quote string into the variable `statement_count_by_category`. Please use the column `sentiment` in the `COUNT` function and give it a new name `count_sentiment`.


```python
# Replace all None
### BEGIN SOLUTION - DO NOT delete this comment for grading purposes
statement_count_by_category = """
SELECT product_category, COUNT(sentiment) AS count_sentiment
FROM reviews
GROUP BY product_category 
ORDER BY count_sentiment DESC
"""
### END SOLUTION - DO NOT delete this comment for grading purposes
print(statement_count_by_category)
```

    
    SELECT product_category, COUNT(sentiment) AS count_sentiment
    FROM reviews
    GROUP BY product_category 
    ORDER BY count_sentiment DESC
    


Query data in Amazon Athena database passing the prepared SQL statement:


```python
%%time
df_count_by_category = wr.athena.read_sql_query(
    sql=statement_count_by_category,
    database=database_name
)

df_count_by_category

# EXPECTED OUTPUT
# Dresses: 6145
# Knits: 4626
# Blouses: 2983
# Sweaters: 1380
# Pants: 1350
# ...
```

    CPU times: user 315 ms, sys: 15.6 ms, total: 330 ms
    Wall time: 3.25 s





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>product_category</th>
      <th>count_sentiment</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Dresses</td>
      <td>6145</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Knits</td>
      <td>4626</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Blouses</td>
      <td>2983</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Sweaters</td>
      <td>1380</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Pants</td>
      <td>1350</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Jeans</td>
      <td>1104</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Fine gauge</td>
      <td>1059</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Skirts</td>
      <td>903</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Jackets</td>
      <td>683</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Lounge</td>
      <td>669</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Swim</td>
      <td>332</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Outerwear</td>
      <td>319</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Shorts</td>
      <td>304</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Sleep</td>
      <td>214</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Legwear</td>
      <td>158</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Intimates</td>
      <td>147</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Layering</td>
      <td>132</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Trend</td>
      <td>118</td>
    </tr>
  </tbody>
</table>
</div>



<a name='c1w1-3.3.'></a>
### 3.3. Which product categories are highest rated by average sentiment?

Set the SQL statement to find the average sentiment per product category, showing the results in the descending order:


```python
statement_avg_by_category = """
SELECT product_category, AVG(sentiment) AS avg_sentiment
FROM {} 
GROUP BY product_category 
ORDER BY avg_sentiment DESC
""".format(table_name)

print(statement_avg_by_category)
```

    
    SELECT product_category, AVG(sentiment) AS avg_sentiment
    FROM reviews 
    GROUP BY product_category 
    ORDER BY avg_sentiment DESC
    


Query data in Amazon Athena database passing the prepared SQL statement:


```python
%%time
df_avg_by_category = wr.athena.read_sql_query(
    sql=statement_avg_by_category,
    database=database_name
)
```

    CPU times: user 401 ms, sys: 12.3 ms, total: 414 ms
    Wall time: 3.63 s


Preview the query results in the temporary S3 bucket:  `s3://aws-athena-query-results-ACCOUNT-REGION/`

**Instructions**: 
- open the link
- check the name of the S3 bucket
- briefly check the content of it


```python
from IPython.core.display import display, HTML

display(HTML('<b>Review <a target="top" href="https://s3.console.aws.amazon.com/s3/buckets/aws-athena-query-results-{}-{}?region={}">Amazon S3 buckets</a></b>'.format(account_id, region, region)))
```


<b>Review <a target="top" href="https://s3.console.aws.amazon.com/s3/buckets/aws-athena-query-results-<bound method Session.account_id of <sagemaker.session.Session object at 0x7faf4a612bd0>>-us-east-1?region=us-east-1">Amazon S3 buckets</a></b>


Preview the results of the query:


```python
df_avg_by_category
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>product_category</th>
      <th>avg_sentiment</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Layering</td>
      <td>0.780303</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Jeans</td>
      <td>0.746377</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Lounge</td>
      <td>0.745889</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Sleep</td>
      <td>0.710280</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Shorts</td>
      <td>0.707237</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Pants</td>
      <td>0.705185</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Intimates</td>
      <td>0.700680</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Jackets</td>
      <td>0.699854</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Skirts</td>
      <td>0.696567</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Legwear</td>
      <td>0.696203</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Fine gauge</td>
      <td>0.692162</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Outerwear</td>
      <td>0.683386</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Knits</td>
      <td>0.653913</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Swim</td>
      <td>0.644578</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Dresses</td>
      <td>0.643287</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Sweaters</td>
      <td>0.641304</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Blouses</td>
      <td>0.641301</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Trend</td>
      <td>0.483051</td>
    </tr>
  </tbody>
</table>
</div>



**Visualization**


```python
def show_values_barplot(axs, space):
    def _show_on_plot(ax):
        for p in ax.patches:
            _x = p.get_x() + p.get_width() + float(space)
            _y = p.get_y() + p.get_height()
            value = round(float(p.get_width()),2)
            ax.text(_x, _y, value, ha="left")

    if isinstance(axs, np.ndarray):
        for idx, ax in np.ndenumerate(axs):
            _show_on_plot(ax)
    else:
        _show_on_plot(axs)
```


```python
# Create plot
barplot = sns.barplot(
    data = df_avg_by_category, 
    y='product_category',
    x='avg_sentiment', 
    color="b", 
    saturation=1
)

# Set the size of the figure
sns.set(rc={'figure.figsize':(15.0, 10.0)})
    
# Set title and x-axis ticks 
plt.title('Average sentiment by product category')
#plt.xticks([-1, 0, 1], ['Negative', 'Neutral', 'Positive'])

# Helper code to show actual values afters bars 
show_values_barplot(barplot, 0.1)

plt.xlabel("Average sentiment")
plt.ylabel("Product category")

plt.tight_layout()
# Do not change the figure name - it is used for grading purposes!
plt.savefig('avg_sentiment_per_category.png', dpi=300)

# Show graphic
plt.show(barplot)
```


![png](output_86_0.png)



```python
# Upload image to S3 bucket
sess.upload_data(path='avg_sentiment_per_category.png', bucket=bucket, key_prefix="images")
```




    's3://sagemaker-us-east-1-139279355515/images/avg_sentiment_per_category.png'



Review the bucket on the account.

**Instructions**: 
- open the link
- click on the S3 bucket name `sagemaker-us-east-1-ACCOUNT`
- open the images folder
- check the existence of the image `avg_sentiment_per_category.png`
- if you click on the image name, you can see the information about the image file. You can also download the file with the command on the top right Object Actions -> Download / Download as
<img src="images/download_image_file.png" width="100%">


```python
from IPython.core.display import display, HTML

display(HTML('<b>Review <a target="top" href="https://s3.console.aws.amazon.com/s3/home?region={}">Amazon S3 buckets</a></b>'.format(region)))
```


<b>Review <a target="top" href="https://s3.console.aws.amazon.com/s3/home?region=us-east-1">Amazon S3 buckets</a></b>


<a name='c1w1-3.4.'></a>
### 3.4. Which product categories have the most reviews?

Set the SQL statement to find the count of sentiment per product category, showing the results in the descending order:


```python
statement_count_by_category_desc = """
SELECT product_category, COUNT(*) AS count_reviews 
FROM {}
GROUP BY product_category 
ORDER BY count_reviews DESC
""".format(table_name)

print(statement_count_by_category_desc)
```

    
    SELECT product_category, COUNT(*) AS count_reviews 
    FROM reviews
    GROUP BY product_category 
    ORDER BY count_reviews DESC
    


Query data in Amazon Athena database passing the prepared SQL statement:


```python
%%time
df_count_by_category_desc = wr.athena.read_sql_query(
    sql=statement_count_by_category_desc,
    database=database_name
)
```

    CPU times: user 284 ms, sys: 12 ms, total: 296 ms
    Wall time: 3.68 s


Store maximum number of sentiment for the visualization plot:


```python
max_sentiment = df_count_by_category_desc['count_reviews'].max()
print('Highest number of reviews (in a single category): {}'.format(max_sentiment))
```

    Highest number of reviews (in a single category): 6145


**Visualization**

<a name='c1w1-ex-4'></a>
### Exercise 4

Use `barplot` function to plot number of reviews per product category.

**Instructions**: Use the `barplot` chart example in the previous section, passing the newly defined dataframe `df_count_by_category_desc` with the count of reviews. Here, please put the `product_category` column into the `y` argument.


```python
# Create seaborn barplot
barplot = sns.barplot(
    ### BEGIN SOLUTION - DO NOT delete this comment for grading purposes
    data=df_count_by_category_desc, # Replace None
    y='count_reviews', # Replace None
    x='product_category', # Replace None
    ### END SOLUTION - DO NOT delete this comment for grading purposes
    color="b",
    saturation=1
)

# Set the size of the figure
sns.set(rc={'figure.figsize':(15.0, 10.0)})
    
# Set title
plt.title("Number of reviews per product category")
plt.xlabel("Number of reviews")
plt.ylabel("Product category")

plt.tight_layout()

# Do not change the figure name - it is used for grading purposes!
plt.savefig('num_reviews_per_category.png', dpi=300)

# Show the barplot
plt.show(barplot)
```


![png](output_98_0.png)



```python
# Upload image to S3 bucket
sess.upload_data(path='num_reviews_per_category.png', bucket=bucket, key_prefix="images")
```




    's3://sagemaker-us-east-1-139279355515/images/num_reviews_per_category.png'



<a name='c1w1-3.5.'></a>
### 3.5. What is the breakdown of sentiments per product category?

Set the SQL statement to find the count of sentiment per product category and sentiment:


```python
statement_count_by_category_and_sentiment = """
SELECT product_category,
         sentiment,
         COUNT(*) AS count_reviews
FROM {}
GROUP BY  product_category, sentiment
ORDER BY  product_category ASC, sentiment DESC, count_reviews
""".format(table_name)

print(statement_count_by_category_and_sentiment)
```

    
    SELECT product_category,
             sentiment,
             COUNT(*) AS count_reviews
    FROM reviews
    GROUP BY  product_category, sentiment
    ORDER BY  product_category ASC, sentiment DESC, count_reviews
    


Query data in Amazon Athena database passing the prepared SQL statement:


```python
%%time
df_count_by_category_and_sentiment = wr.athena.read_sql_query(
    sql=statement_count_by_category_and_sentiment,
    database=database_name
)
```

    CPU times: user 238 ms, sys: 31.3 ms, total: 269 ms
    Wall time: 2.89 s



```python
df_count_by_category_and_sentiment
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>product_category</th>
      <th>sentiment</th>
      <th>count_reviews</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Blouses</td>
      <td>1</td>
      <td>2256</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Blouses</td>
      <td>0</td>
      <td>384</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Blouses</td>
      <td>-1</td>
      <td>343</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Dresses</td>
      <td>1</td>
      <td>4634</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Dresses</td>
      <td>0</td>
      <td>830</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Dresses</td>
      <td>-1</td>
      <td>681</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Fine gauge</td>
      <td>1</td>
      <td>837</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Fine gauge</td>
      <td>0</td>
      <td>118</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Fine gauge</td>
      <td>-1</td>
      <td>104</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Intimates</td>
      <td>1</td>
      <td>117</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Intimates</td>
      <td>0</td>
      <td>16</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Intimates</td>
      <td>-1</td>
      <td>14</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Jackets</td>
      <td>1</td>
      <td>550</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Jackets</td>
      <td>0</td>
      <td>61</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Jackets</td>
      <td>-1</td>
      <td>72</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Jeans</td>
      <td>1</td>
      <td>909</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Jeans</td>
      <td>0</td>
      <td>110</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Jeans</td>
      <td>-1</td>
      <td>85</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Knits</td>
      <td>1</td>
      <td>3523</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Knits</td>
      <td>0</td>
      <td>605</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Knits</td>
      <td>-1</td>
      <td>498</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Layering</td>
      <td>1</td>
      <td>113</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Layering</td>
      <td>0</td>
      <td>9</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Layering</td>
      <td>-1</td>
      <td>10</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Legwear</td>
      <td>1</td>
      <td>126</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Legwear</td>
      <td>0</td>
      <td>16</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Legwear</td>
      <td>-1</td>
      <td>16</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Lounge</td>
      <td>1</td>
      <td>545</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Lounge</td>
      <td>0</td>
      <td>78</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Lounge</td>
      <td>-1</td>
      <td>46</td>
    </tr>
    <tr>
      <th>30</th>
      <td>Outerwear</td>
      <td>1</td>
      <td>254</td>
    </tr>
    <tr>
      <th>31</th>
      <td>Outerwear</td>
      <td>0</td>
      <td>29</td>
    </tr>
    <tr>
      <th>32</th>
      <td>Outerwear</td>
      <td>-1</td>
      <td>36</td>
    </tr>
    <tr>
      <th>33</th>
      <td>Pants</td>
      <td>1</td>
      <td>1074</td>
    </tr>
    <tr>
      <th>34</th>
      <td>Pants</td>
      <td>0</td>
      <td>154</td>
    </tr>
    <tr>
      <th>35</th>
      <td>Pants</td>
      <td>-1</td>
      <td>122</td>
    </tr>
    <tr>
      <th>36</th>
      <td>Shorts</td>
      <td>1</td>
      <td>240</td>
    </tr>
    <tr>
      <th>37</th>
      <td>Shorts</td>
      <td>0</td>
      <td>39</td>
    </tr>
    <tr>
      <th>38</th>
      <td>Shorts</td>
      <td>-1</td>
      <td>25</td>
    </tr>
    <tr>
      <th>39</th>
      <td>Skirts</td>
      <td>1</td>
      <td>714</td>
    </tr>
    <tr>
      <th>40</th>
      <td>Skirts</td>
      <td>0</td>
      <td>104</td>
    </tr>
    <tr>
      <th>41</th>
      <td>Skirts</td>
      <td>-1</td>
      <td>85</td>
    </tr>
    <tr>
      <th>42</th>
      <td>Sleep</td>
      <td>1</td>
      <td>175</td>
    </tr>
    <tr>
      <th>43</th>
      <td>Sleep</td>
      <td>0</td>
      <td>16</td>
    </tr>
    <tr>
      <th>44</th>
      <td>Sleep</td>
      <td>-1</td>
      <td>23</td>
    </tr>
    <tr>
      <th>45</th>
      <td>Sweaters</td>
      <td>1</td>
      <td>1036</td>
    </tr>
    <tr>
      <th>46</th>
      <td>Sweaters</td>
      <td>0</td>
      <td>193</td>
    </tr>
    <tr>
      <th>47</th>
      <td>Sweaters</td>
      <td>-1</td>
      <td>151</td>
    </tr>
    <tr>
      <th>48</th>
      <td>Swim</td>
      <td>1</td>
      <td>252</td>
    </tr>
    <tr>
      <th>49</th>
      <td>Swim</td>
      <td>0</td>
      <td>42</td>
    </tr>
    <tr>
      <th>50</th>
      <td>Swim</td>
      <td>-1</td>
      <td>38</td>
    </tr>
    <tr>
      <th>51</th>
      <td>Trend</td>
      <td>1</td>
      <td>78</td>
    </tr>
    <tr>
      <th>52</th>
      <td>Trend</td>
      <td>0</td>
      <td>19</td>
    </tr>
    <tr>
      <th>53</th>
      <td>Trend</td>
      <td>-1</td>
      <td>21</td>
    </tr>
  </tbody>
</table>
</div>



Prepare for stacked percentage horizontal bar plot showing proportion of sentiments per product category.


```python
# Create grouped dataframes by category and by sentiment
grouped_category = df_count_by_category_and_sentiment.groupby('product_category')
grouped_star = df_count_by_category_and_sentiment.groupby('sentiment')

# Create sum of sentiments per star sentiment
df_sum = df_count_by_category_and_sentiment.groupby(['sentiment']).sum()

# Calculate total number of sentiments
total = df_sum['count_reviews'].sum()
print('Total number of reviews: {}'.format(total))
```

    Total number of reviews: 22626


Create dictionary of product categories and array of star rating distribution per category.


```python
distribution = {}
count_reviews_per_star = []
i=0

for category, sentiments in grouped_category:
    count_reviews_per_star = []
    for star in sentiments['sentiment']:
        count_reviews_per_star.append(sentiments.at[i, 'count_reviews'])
        i=i+1;
    distribution[category] = count_reviews_per_star
```

Build array per star across all categories.


```python
distribution
```




    {'Blouses': [2256, 384, 343],
     'Dresses': [4634, 830, 681],
     'Fine gauge': [837, 118, 104],
     'Intimates': [117, 16, 14],
     'Jackets': [550, 61, 72],
     'Jeans': [909, 110, 85],
     'Knits': [3523, 605, 498],
     'Layering': [113, 9, 10],
     'Legwear': [126, 16, 16],
     'Lounge': [545, 78, 46],
     'Outerwear': [254, 29, 36],
     'Pants': [1074, 154, 122],
     'Shorts': [240, 39, 25],
     'Skirts': [714, 104, 85],
     'Sleep': [175, 16, 23],
     'Sweaters': [1036, 193, 151],
     'Swim': [252, 42, 38],
     'Trend': [78, 19, 21]}




```python
df_distribution_pct = pd.DataFrame(distribution).transpose().apply(
    lambda num_sentiments: num_sentiments/sum(num_sentiments)*100, axis=1
)
df_distribution_pct.columns=['1', '0', '-1']
df_distribution_pct
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>1</th>
      <th>0</th>
      <th>-1</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Blouses</th>
      <td>75.628562</td>
      <td>12.872947</td>
      <td>11.498491</td>
    </tr>
    <tr>
      <th>Dresses</th>
      <td>75.410903</td>
      <td>13.506916</td>
      <td>11.082181</td>
    </tr>
    <tr>
      <th>Fine gauge</th>
      <td>79.036827</td>
      <td>11.142587</td>
      <td>9.820585</td>
    </tr>
    <tr>
      <th>Intimates</th>
      <td>79.591837</td>
      <td>10.884354</td>
      <td>9.523810</td>
    </tr>
    <tr>
      <th>Jackets</th>
      <td>80.527086</td>
      <td>8.931186</td>
      <td>10.541728</td>
    </tr>
    <tr>
      <th>Jeans</th>
      <td>82.336957</td>
      <td>9.963768</td>
      <td>7.699275</td>
    </tr>
    <tr>
      <th>Knits</th>
      <td>76.156507</td>
      <td>13.078253</td>
      <td>10.765240</td>
    </tr>
    <tr>
      <th>Layering</th>
      <td>85.606061</td>
      <td>6.818182</td>
      <td>7.575758</td>
    </tr>
    <tr>
      <th>Legwear</th>
      <td>79.746835</td>
      <td>10.126582</td>
      <td>10.126582</td>
    </tr>
    <tr>
      <th>Lounge</th>
      <td>81.464873</td>
      <td>11.659193</td>
      <td>6.875934</td>
    </tr>
    <tr>
      <th>Outerwear</th>
      <td>79.623824</td>
      <td>9.090909</td>
      <td>11.285266</td>
    </tr>
    <tr>
      <th>Pants</th>
      <td>79.555556</td>
      <td>11.407407</td>
      <td>9.037037</td>
    </tr>
    <tr>
      <th>Shorts</th>
      <td>78.947368</td>
      <td>12.828947</td>
      <td>8.223684</td>
    </tr>
    <tr>
      <th>Skirts</th>
      <td>79.069767</td>
      <td>11.517165</td>
      <td>9.413068</td>
    </tr>
    <tr>
      <th>Sleep</th>
      <td>81.775701</td>
      <td>7.476636</td>
      <td>10.747664</td>
    </tr>
    <tr>
      <th>Sweaters</th>
      <td>75.072464</td>
      <td>13.985507</td>
      <td>10.942029</td>
    </tr>
    <tr>
      <th>Swim</th>
      <td>75.903614</td>
      <td>12.650602</td>
      <td>11.445783</td>
    </tr>
    <tr>
      <th>Trend</th>
      <td>66.101695</td>
      <td>16.101695</td>
      <td>17.796610</td>
    </tr>
  </tbody>
</table>
</div>



**Visualization**

Plot the distributions of sentiments per product category.


```python
categories = df_distribution_pct.index

# Plot bars
plt.figure(figsize=(10,5))

df_distribution_pct.plot(kind="barh", 
                         stacked=True, 
                         edgecolor='white',
                         width=1.0,
                         color=['green', 
                                'orange', 
                                'blue'])

plt.title("Distribution of reviews per sentiment per category", 
          fontsize='16')

plt.legend(bbox_to_anchor=(1.04,1), 
           loc="upper left",
           labels=['Positive', 
                   'Neutral', 
                   'Negative'])

plt.xlabel("% Breakdown of sentiments", fontsize='14')
plt.gca().invert_yaxis()
plt.tight_layout()

# Do not change the figure name - it is used for grading purposes!
plt.savefig('distribution_sentiment_per_category.png', dpi=300)
plt.show()
```


    <Figure size 720x360 with 0 Axes>



![png](output_114_1.png)



```python
# Upload image to S3 bucket
sess.upload_data(path='distribution_sentiment_per_category.png', bucket=bucket, key_prefix="images")
```




    's3://sagemaker-us-east-1-139279355515/images/distribution_sentiment_per_category.png'



<a name='c1w1-3.6.'></a>
### 3.6. Analyze the distribution of review word counts

Set the SQL statement to count the number of the words in each of the reviews:


```python
statement_num_words = """
    SELECT CARDINALITY(SPLIT(review_body, ' ')) as num_words
    FROM {}
""".format(table_name)

print(statement_num_words)
```

    
        SELECT CARDINALITY(SPLIT(review_body, ' ')) as num_words
        FROM reviews
    


Query data in Amazon Athena database passing the SQL statement:


```python
%%time
df_num_words = wr.athena.read_sql_query(
    sql=statement_num_words,
    database=database_name
)
```

    CPU times: user 386 ms, sys: 19.1 ms, total: 405 ms
    Wall time: 3.4 s



```python
df_num_words
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>num_words</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>70</td>
    </tr>
    <tr>
      <th>1</th>
      <td>68</td>
    </tr>
    <tr>
      <th>2</th>
      <td>102</td>
    </tr>
    <tr>
      <th>3</th>
      <td>27</td>
    </tr>
    <tr>
      <th>4</th>
      <td>36</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th>22621</th>
      <td>28</td>
    </tr>
    <tr>
      <th>22622</th>
      <td>40</td>
    </tr>
    <tr>
      <th>22623</th>
      <td>44</td>
    </tr>
    <tr>
      <th>22624</th>
      <td>90</td>
    </tr>
    <tr>
      <th>22625</th>
      <td>21</td>
    </tr>
  </tbody>
</table>
<p>22626 rows × 1 columns</p>
</div>



Print out and analyse some descriptive statistics: 


```python
summary = df_num_words["num_words"].describe(percentiles=[0.10, 0.20, 0.30, 0.40, 0.50, 0.60, 0.70, 0.80, 0.90, 1.00])
summary
```




    count    22626.000000
    mean        62.709847
    std         29.993735
    min          2.000000
    10%         22.000000
    20%         33.000000
    30%         42.000000
    40%         51.000000
    50%         61.000000
    60%         72.000000
    70%         86.000000
    80%         97.000000
    90%        103.000000
    100%       122.000000
    max        122.000000
    Name: num_words, dtype: float64



Plot the distribution of the words number per review:


```python
df_num_words["num_words"].plot.hist(xticks=[0, 16, 32, 64, 128, 256], bins=100, range=[0, 256]).axvline(
    x=summary["100%"], c="red"
)

plt.xlabel("Words number", fontsize='14')
plt.ylabel("Frequency", fontsize='14')
plt.savefig('distribution_num_words_per_review.png', dpi=300)
plt.show()
```


![png](output_125_0.png)



```python
# Upload image to S3 bucket
sess.upload_data(path='distribution_num_words_per_review.png', bucket=bucket, key_prefix="images")
```




    's3://sagemaker-us-east-1-139279355515/images/distribution_num_words_per_review.png'



Upload the notebook into S3 bucket for grading purposes.

**Note**: you may need to click on "Save" button before the upload.


```python
!aws s3 cp ./C1_W1_Assignment.ipynb s3://$bucket/C1_W1_Assignment_Learner.ipynb
```

    upload: ./C1_W1_Assignment.ipynb to s3://sagemaker-us-east-1-139279355515/C1_W1_Assignment_Learner.ipynb


Please go to the main lab window and click on `Submit` button (see the `Finish the lab` section of the instructions).


```python

```
