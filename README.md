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
