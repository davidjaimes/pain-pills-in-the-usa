# Data Collection

Source: [California Opioid Overdose Surveillance Dashboard](https://discovery.cdph.ca.gov/CDIC/ODdash/)

# Data Restructure

```Python
import pandas as pd
import glob as gl
files = sorted(gl.glob('data/*.csv'))

with open('death-rates.csv', 'a') as fname:
    for f in files:
        print(f)
        df = pd.read_csv(f, skiprows=2)
        boolean = []
        for z in df['Zip Code']:
            try:
                int(z)
                boolean.append(True)
            except ValueError:
                boolean.append(False)
        df = df[boolean]
        df.to_csv(fname, index=False, columns=['Zip Code', 'Rates'], header=False)
```