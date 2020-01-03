# Data Collection

We went to this [Washington Post investigation](https://www.washingtonpost.com/graphics/2019/investigations/dea-pain-pill-database/#download-resources) and got the state-wide data for California. The file `arcos-ca-statewide-itemized.tsv` is 6.4 GB and came from the Drug Enforcement Administration's (DEA) database known as ARCOS (Automation of Reports and Consolidated Orders System). The column headers for this file are :

```Python
import pandas as pd
df = pd.read_csv('arcos-ca-statewide-itemized.tsv', delimiter='\t', nrows=10)
print(df.columns)
```

```
Index(['REPORTER_DEA_NO', 'REPORTER_BUS_ACT', 'REPORTER_NAME',
       'REPORTER_ADDL_CO_INFO', 'REPORTER_ADDRESS1', 'REPORTER_ADDRESS2',
       'REPORTER_CITY', 'REPORTER_STATE', 'REPORTER_ZIP', 'REPORTER_COUNTY',
       'BUYER_DEA_NO', 'BUYER_BUS_ACT', 'BUYER_NAME', 'BUYER_ADDL_CO_INFO',
       'BUYER_ADDRESS1', 'BUYER_ADDRESS2', 'BUYER_CITY', 'BUYER_STATE',
       'BUYER_ZIP', 'BUYER_COUNTY', 'TRANSACTION_CODE', 'DRUG_CODE', 'NDC_NO',
       'DRUG_NAME', 'QUANTITY', 'UNIT', 'ACTION_INDICATOR', 'ORDER_FORM_NO',
       'CORRECTION_NO', 'STRENGTH', 'TRANSACTION_DATE', 'CALC_BASE_WT_IN_GM',
       'DOSAGE_UNIT', 'TRANSACTION_ID', 'Product_Name', 'Ingredient_Name',
       'Measure', 'MME_Conversion_Factor', 'Combined_Labeler_Name',
       'Revised_Company_Name', 'Reporter_family', 'dos_str'],
      dtype='object')
```

# Data Pruning

We did not need all 42 columns and only chose to work with these 4 column names:

```
Index(['BUYER_CITY', 'BUYER_ZIP', TRANSACTION_DATE', 'DOSAGE_UNIT', Ingredient_Name'],
      dtype='object')
```

We took advantage of the Pandas `chuncksize` parameter to parse through the 6.4 GB file and save it as multiple files, each with 200,000 rows. We turned the **6.4 GB** file and reduced it down to **943 MB** with the following Python code:

```Python
import pandas as pd
path = 'arcos-ca-statewide-itemized.tsv'
chunck = pd.read_csv(path, delimiter='\t', chunksize=200000, low_memory=False)
for i, c in enumerate(chunck):
    c = c[['BUYER_CITY', 'BUYER_ZIP', 'TRANSACTION_DATE', 'DOSAGE_UNIT', 'Ingredient_Name']]
    c.to_csv(f'arcos-ca/data/statewide-{i+1:02d}.csv', index=False)
```

# Zip Codes

We found a total of 1,288 zip codes with the following code:

```Python
import glob as gl
import pandas as pd
import numpy as np
files = sorted(gl.glob('arcos-ca/data/*.csv'))
df = pd.read_csv(files[0])
unique, indices = np.unique(df['BUYER_ZIP'], return_index=True)
cities = df['BUYER_CITY'][indices]
for f in files[1:]:
    df = pd.read_csv(f)
    unq, inx = np.unique(df['BUYER_ZIP'], return_index=True)
    concat = np.concatenate((unique, unq), axis=None)
    cities = np.concatenate((cities, df['BUYER_CITY'][inx]), axis=None)
    unique, indices = np.unique(concat, return_index=True)
    cities = cities[indices]
df = pd.DataFrame({'Zip Code': unique, 'City': cities})
df = df.sort_values(by=['Zip Code'])
df.to_csv('arcos-ca/zip_codes.csv', index=False)
```

# Transactions Per Zip Code

The code to find the total number of transactions per zip code:

```Python
import glob as gl
import pandas as pd
zipcode = pd.read_csv('arcos-ca/zip_codes.csv')
zipcode['Counts'] = 0
files = sorted(gl.glob('arcos-ca/data/*.csv'))
for i, f in enumerate(files):
    df = pd.read_csv(f)
    # Uncomment next two lines for specific year.
    # df['TRANSACTION_DATE'] = pd.to_datetime(df['TRANSACTION_DATE'], format='%m%d%Y')
    # df = df[df['TRANSACTION_DATE'].dt.year == 2010]
    valcnt = pd.DataFrame(df['BUYER_ZIP'].value_counts()).reset_index()
    valcnt = valcnt.rename(columns={'index': 'Zip Code', 'BUYER_ZIP': 'Counts'})
    zipcode = pd.merge(zipcode, valcnt, how='left', on='Zip Code')
    zipcode = zipcode.fillna(0)
    zipcode['Counts_x'] = zipcode['Counts_x'] + zipcode['Counts_y']
    zipcode = zipcode.rename(columns={'Counts_x': 'Counts'})
    del zipcode['Counts_y']
zipcode = zipcode.sort_values(by=['Counts'], ascending=False)
zipcode = zipcode.rename(columns={'Counts': 'Transactions'})
zipcode['Transactions'] = zipcode['Transactions'].astype('int32')
zipcode.to_csv('arcos-ca/transactions.csv', index=False)
````

## Top Ten

Zip Code|City|Transactions
---|---|---
95350|MODESTO|67935
90242|DOWNEY|61370
95355|MODESTO|54514
93003|VENTURA|51060
95630|FOLSOM|50943
92021|EL CAJON|49944
92307|APPLE VALLEY|47821
93065|SIMI VALLEY|46860
95370|SONORA|45992
93720|FRESNO|45902

# Pills Per Zip code

The code to find the total number of pills per zip code:

```Python
import pandas as pd
import glob as gl
merge = pd.read_csv('arcos-ca/zip_codes.csv')
merge['DOSAGE_UNIT'] = 0
merge = merge.rename(columns={'Zip Code': 'BUYER_ZIP'})
files = sorted(gl.glob('arcos-ca/data/*.csv'))
for i, f in enumerate(files):
    df = pd.read_csv(f)
    # Uncomment next two lines for specific year.
    # df['TRANSACTION_DATE'] = pd.to_datetime(df['TRANSACTION_DATE'], format='%m%d%Y')
    # df = df[df['TRANSACTION_DATE'].dt.year == 2010]
    del df['TRANSACTION_DATE']
    grp = df.groupby('BUYER_ZIP')
    merge = pd.merge(merge, grp.sum(), how='left', on='BUYER_ZIP')
    merge = merge.fillna(0)
    merge['DOSAGE_UNIT_x'] = merge['DOSAGE_UNIT_x'] + merge['DOSAGE_UNIT_y']
    merge = merge.rename(columns={'DOSAGE_UNIT_x': 'DOSAGE_UNIT'})
    del merge['DOSAGE_UNIT_y']
merge = merge.rename(columns={'BUYER_ZIP': 'Zip Code', 'DOSAGE_UNIT': 'Pills'})
merge = merge.sort_values(by=['Pills'], ascending=False)
merge['Pills'] = merge['Pills'].astype('int32')
merge.to_csv('arcos-ca/pills.csv', index=False)
```

## Top Ten

Zip Code|City|Pills
---|---|---
94550|LIVERMORE|368045810
90242|DOWNEY|187157905
92010|CARLSBAD|74056390
95350|MODESTO|42161712
95825|SACRAMENTO|41336891
95355|MODESTO|39931344
95926|CHICO|37066200
96001|REDDING|34265364
95823|SACRAMENTO|34035590
93534|LANCASTER|32954470

