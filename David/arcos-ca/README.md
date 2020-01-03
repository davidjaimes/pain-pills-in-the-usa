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
Index(['BUYER_ZIP', TRANSACTION_DATE', 'DOSAGE_UNIT', Ingredient_Name'],
      dtype='object')
```

We took advantage of the Pandas `chuncksize` parameter to parse through the 6.4 GB file and save it as multiple files, each with 200,000 rows. We turned the **6.4 GB** file and reduced it down to **795 MB** with the following Python code:

```Python
import pandas as pd
path = 'arcos-ca-statewide-itemized.tsv'
chunck = pd.read_csv(path, delimiter='\t', chunksize=200000, low_memory=False)
for i, c in enumerate(chunck):
    c = c[['BUYER_ZIP', 'TRANSACTION_DATE', 'DOSAGE_UNIT', 'Ingredient_Name']]
    c.to_csv(f'arcos-ca/data/statewide-{i+1:03d}.csv', index=False)
```

# Zip Codes

We found a total of 1,288 zip codes with the following code:

```Python
import glob as gl
import pandas as pd
import numpy as np
files = sorted(gl.glob('arcos-ca/data/*.csv'))
df = pd.read_csv(files[0])
unique = df['BUYER_ZIP'].unique()
for f in files[1:]:
    df = pd.read_csv(f)
    concat = np.concatenate((unique, df['BUYER_ZIP'].unique()), axis=None)
    unique = np.unique(concat)
np.savetxt('arcos-ca/zip_codes.txt', sorted(unique), fmt='%9d')
```

# Transactions Per Zip Code

The code to find the total number of transactions per zip code:

```Python
import glob as gl
import pandas as pd
zipcode = pd.read_table('arcos-ca/zip_codes.txt', names=['Zip Code'])
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
zipcode = zipcode.rename(columns={'Counts': 'Transactions'}).astype('int32')
zipcode.to_csv('arcos-ca/transactions.csv', index=False)
````

## Top Ten

Index|Zip Code|Transactions|City
---|---|---|---
0|95350|67935|Modesto
1|90242|61370|Downey
2|95355|54514|Modesto
3|93003|51060|Ventura
4|95630|50943|Folsom
5|92021|49944|El Cajon
6|92307|47821|Apple Valley
7|93065|46860|Simi Valley
8|95370|45992|Sonora
9|93720|45902|Fresno

# Pills Per Zip code

The code to find the total number of pills per zip code:

```Python
import pandas as pd
import glob as gl
merge = pd.read_table('arcos-ca/zip_codes.txt', names=['BUYER_ZIP'])
merge['DOSAGE_UNIT'] = 0
merge = merge.set_index('BUYER_ZIP')
files = sorted(gl.glob('arcos-ca/data/*.csv'))
for i, f in enumerate(files):
    df = pd.read_csv(f, usecols=['BUYER_ZIP', 'DOSAGE_UNIT', 'TRANSACTION_DATE'])
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
merge = merge.reset_index()
merge = merge.rename(columns={'BUYER_ZIP': 'Zip Code', 'DOSAGE_UNIT': 'Pills'})
merge = merge.sort_values(by=['Pills'], ascending=False).astype('int32')
merge.to_csv('arcos-ca/pills.csv', index=False)
```

## Top Ten

Index|Zip Code|Pills|City
---|---|---|---
0|94550|368045810|Livermore
1|90242|187157905|Downey
2|92010|74056390|Carlsbad
3|95350|42161712|Modesto
4|95825|41336891|Sacramento
5|95355|39931344|Modesto
6|95926|37066200|Chico
7|96001|34265364|Redding
8|95823|34035590|Sacramento
9|93534|32954470|Lancaster
