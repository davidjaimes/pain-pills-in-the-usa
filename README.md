### Data Collection

We went to this [Washington Post](https://www.washingtonpost.com/graphics/2019/investigations/dea-pain-pill-database/#download-resources) investigation and got the state-wide data for California from 2006 to 2012. The file `arcos-ca-statewide-itemized.tsv` is 6.4 GB and came from the Drug Enforcement Administration's (DEA) database known as ARCOS (Automation of Reports and Consolidated Orders System).

### Data Pruning

We did not need all 42 columns and only chose to work with these 12 column names:
```
Index(['BUYER_CITY', 'BUYER_STATE', 'BUYER_ZIP', 'BUYER_COUNTY', 'DRUG_NAME',
       'QUANTITY', 'TRANSACTION_DATE', 'CALC_BASE_WT_IN_GM', 'DOSAGE_UNIT',
       'Product_Name', 'Ingredient_Name', 'Measure'],
      dtype='object')
```

We took advantage of the Pandas `chuncksize` parameter to parse through the 6.4 GB file and save it as multiple files with 200,000 rows with the following Python code:
```Python
import pandas as pd
path = 'arcos-ca-statewide-itemized.tsv'
chunck = pd.read_csv(path, delimiter='\t', encoding='utf-8', chunksize=200000, low_memory=False)
for i, c in enumerate(chunck):
    c = c[['BUYER_CITY', 'BUYER_STATE', 'BUYER_ZIP', 'BUYER_COUNTY',
       'DRUG_NAME', 'QUANTITY', 'TRANSACTION_DATE', 'CALC_BASE_WT_IN_GM',
       'DOSAGE_UNIT', 'Product_Name', 'Ingredient_Name', 'Measure']]
    c.to_csv(f'ca-opioid-data/ca-statewide-{i+1:03d}.csv', encoding='utf-8', index=False)
```

The result turned the 6.4 GB file and saved it to 70 files, all totaling 2.0 GB.

### Unique Zip Codes

Instead of grabbing approximately all 2,500 California zip codes, we wanted to find only the zip codes listed in the 70 files of data. The code to find the unique codes is as follows:
```Python
import glob as gl
import pandas as pd
import numpy as np
files = sorted(gl.glob('ca-opioid-data/*.csv'))
df = pd.read_csv(files[0])
unique = df['BUYER_ZIP'].unique()
for f in files[1:]:
    df = pd.read_csv(f)
    concat = np.concatenate((unique, df['BUYER_ZIP'].unique()), axis=None)
    unique = np.unique(concat)
np.savetxt('unique_zip_codes.txt', sorted(unique), fmt='%9d')
```

We found a total of 1,288 zip codes.
