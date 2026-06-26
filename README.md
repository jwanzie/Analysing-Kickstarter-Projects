### Kickstarter Projects: Data Exploration

### Overview

This document is used to highlight the key issues of data quality that were identified during the cleaning and exploratory phase of the Kickstarter Projects dataset(`ks-projects-201612.csv`) and how they were resolved.

### Data Issues Observed

#### 1. Empty Columns

There were 4 columns (`Unnamed: 13, Unnamed: 14, Unnamed: 15, Unnamed: 16`) in the dataset that contained missing records. Since over 95% of the records were missing, the right call was to drop these columns.

```python
df.drop(columns = ['Unnamed: 13', 'Unnamed: 14', 'Unnamed: 15', 'Unnamed: 16'], inplace = True)
```

#### 2. Unstandardized Name

Given the column `usd pledged` naming does not match naming conventions in SQL and with empty space at the end of each column name. This was fixed by stripping and replacing the empty space within the name with '_'.

```python
df.columns = df.columns.str.strip().str.replace(' ', '_').str.strip('_')
```

#### 3. Missing Values

Three columns contained missing values with the column `name` missing 4 records, `category` missing 5 records and `usd_pledged` missing 3790 records. Since the name records are unique, in order not to drop these missing records, I replaced it with `Unknown`.

```python
df['name'] = df['name'].fillna('Unknown')
```

For the category records, by exploring patterns between the column `category` and `main_category` these columns were filled.

```python
df.loc[36671, 'category'] = 'Webseries'
df.loc[41069, 'category'] = 'Interactive Design'
df.loc[63544, 'category'] = 'Unknown'
df.loc[96753, 'category'] = 'Art'
df.loc[269930, 'category'] = 'Design'
```

For the other 3790 records, using the patterns between the `usd_pledged`, `pledged` and `currency` columns, I replace the corresponding null values which replaced 2733 records while dropping the null values corresponding to other currencies as these constituted ~0.3% of the records.

```python
usd_mask = (df['usd_pledged'].isnull()) & (df['currency'] == 'USD')
df.loc[usd_mask, 'usd_pledged'] = df.loc[usd_mask, 'pledged']

zero_mask = (df['usd_pledged'].isnull()) & (df['pledged'] == '0')
df.loc[zero_mask, 'usd_pledged'] = '0'

df = df.dropna(subset = ['usd_pledged'])
```

#### 4. Misnamed Records

Within the `country` column there are 2945 records named `N,"0` and others as numbers. These records were replaced with `Unknown` so as not to lose the records.

```python
df['country'] = df['country'].apply(lambda x: x if isinstance(x, str) and len(x) == 2 and x.isupper() else 'Unknown')
```

#### 5. Mismatched Values

While parsing dates, it became clear that some records do not match the columns they're in meaning the csv had a missing comma. By creating a `bad_rows` mask based on invalid `deadline` values and deleting those rows.

```python
bad_rows = pd.to_datetime(df['deadline'], errors = 'coerce').isna()

df = df[~bad_rows]
```
