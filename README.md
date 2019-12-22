### Data Collection

We went to this [Washington Post](https://www.washingtonpost.com/graphics/2019/investigations/dea-pain-pill-database/#download-resources) investigation and got the state-wide data for California. The file `arcos-ca-statewide-itemized.tsv` is 6.4 GB and came from the Drug Enforcement Administration's (DEA) database known as ARCOS (Automation of Reports and Consolidated Orders System).

### Data Pruning

We did not need all 42 columns and only chose to work with these 12 column names:
```
Index(['BUYER_CITY', 'BUYER_STATE', 'BUYER_ZIP', 'BUYER_COUNTY', 'DRUG_NAME',
       'QUANTITY', 'TRANSACTION_DATE', 'CALC_BASE_WT_IN_GM', 'DOSAGE_UNIT',
       'Product_Name', 'Ingredient_Name', 'Measure'],
      dtype='object')
```

We took advantage of the `chuncksize` parameter to parse through the 6.4 GB file and save it as multiple files with 50,000 rows each with the following Python code:
```Python
import pandas as pd
path = 'arcos-ca-statewide-itemized.tsv'
chunck = pd.read_csv(path, delimiter='\t', encoding='utf-8', chunksize=60000, low_memory=False)
for i, c in enumerate(chunck):
    c = c[['BUYER_CITY', 'BUYER_STATE', 'BUYER_ZIP', 'BUYER_COUNTY',
       'DRUG_NAME', 'QUANTITY', 'TRANSACTION_DATE', 'CALC_BASE_WT_IN_GM',
       'DOSAGE_UNIT', 'Product_Name', 'Ingredient_Name', 'Measure']]
    print(c.columns)
    c.to_csv(f'chunck/ca-statewide-{i+1:03d}.csv', encoding='utf-8', index=False)
```
