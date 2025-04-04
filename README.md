# The Ultimate Customer Segmentation Analysis

## Task Overview
The objective of this task is to analyze customer data, clean it, extract insights, and generate a report using Python.


## Data Source
The dataset used is the "MOCK_DATA.csv" file which contains detailed customer information with attributes such as Customer ID, Country, Total Purchases, and Sign_up Date.


## Tools
-Excel
-Python


## Steps
These are the various steps involved:

The first step involved importing the necessary python libraries, followed by loading the dataset into a Pandas DataFrame, ensuring only the first 75% of the rows were loaded.

```
# Importing the necessary libraries
import pandas as pd
import numpy as np
from datetime import datetime
```

```
# Read the csv file 

total_rows = pd.read_csv('MOCK_DATA.csv').shape[0]
df = pd.read_csv('MOCK_DATA.csv', nrows = int(0.75*total_rows)) #loading the first 75% of the rows 
print(df)
```

### Data Cleaning

```
# Remove duplicate rows
df.drop_duplicates(keep='first')
```

```
#Convert the Sign-up Date column to a proper datetime format and replace all entries from the year 2020 with NaN
df['Sign-up Date'] = pd.to_datetime(df['Sign-up Date'])
df.loc[df['Sign-up Date'].dt.year == 2020, 'Sign-up Date'] = np.nan
```

```
#drop all rows where the column "Total Purchases" < 5 and from Canada
df = df.drop(df[(df['Total Purchases']<5) & (df['Country'] == "Canada")].index)
```

```
dict = {'Total Purchases': 'Total_Purchases', 'Sign-up Date': 'Sign_up_Date'}
df.rename(columns= dict, inplace = True) # renaming the columns to avoid syntax error based on the space in between these column names and the use of hyphen which does not follow the rules for naming variables in python. Customer ID column is left out because it's not involved in this task.
df.head()
```

### Feature Engineering

```
# create a new column called "Loyalty Score" based on these conditions
def Loyalty_Score(Total_Purchases, Sign_up_Date):
    
    current_date = datetime.now() #In order to calculate the membership duration, we need to know the current date. Then the Sign-up Date will be subtracted from it which will be divided by 365 (365 days = a year)
    Membership_duration = (current_date - Sign_up_Date).days / 365 
   
    if Total_Purchases > 20:
        return 3
    elif 10 <= Total_Purchases <= 20:
        return 2
    elif Total_Purchases < 10:
        if Membership_duration > 3: #Sign-up Date is more than 3 years
            return 2
        else:
            return 1
```

```
# This is to apply the Loyalty_Score to each row having met the above conditions based on the values in the 2 columns involved
df['Loyalty_Score'] = df.apply(lambda row: Loyalty_Score(row['Total_Purchases'], row['Sign_up_Date']), axis=1)
df.head()
```

### Data Aggregation & Filtering

```
df_filtered = df.drop(df[df['Country'] == 'Mexico'].index) # drop all rows with Mexico (Country)

# GroupBy Country and calculate average and maximum of Total Purchases by Country
agg_df = df_filtered.groupby('Country')['Total_Purchases'].agg(['mean', 'max']).sort_values(by=['mean', 'max'], ascending=False) 
print(agg_df)
```

```
# calculate the most common Loyalty Score across countries
Mode_df = df_filtered.groupby('Country')['Loyalty_Score'].apply(lambda x:x.mode().iloc[0])
Mode_df = Mode_df.sort_values(ascending=False)

print(Mode_df)
```

### Data Export & Report Generation
```
# Save the date to a new CSV file
df_filtered.to_csv('processed_data_17_03_2025.csv', index=False)

df_filtered.sample(10) # print the first 10 random rows of the processed DataFrame
```

### Insights
Key insights drawn from the analysis revealed that customers from **Australia** had the highest mean value of **47**, and also, the highest maximum Total Purchases of **47** value. The most common Loyalty Score across countries was **3**, this implies that a great number of the customers across diverse countries had more than 20 Total Purchases.
