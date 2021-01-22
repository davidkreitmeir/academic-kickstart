---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Automate the boring stuff: SQL Query building for WRDS"
subtitle: "A simple python program to automate the SQL query building for Wharton's Reasearch Data Service [WRDS]"
summary: ""
authors: []
tags: []
categories: []
date: 2021-01-22T17:30:21+11:00
lastmod: 2021-01-22T17:30:21+11:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

The following is a simple `python` program to automate the SQL query building in order to extract data from Wharton's Research Data Service [WRDS](https://wrds-www.wharton.upenn.edu/).

## First things first
First, we need to load the library and connect to the WRDS database. This is easiest when using `Jupyter Notebooks`. WRDS provides a nice tutorial on how to connect to WRDS from `Jupyter Notebooks`, which can be found here:  
https://wrds-www.wharton.upenn.edu/documents/1443/wrds_connection.html

## Now let's build some queries

Simplified, SQL queries take the general form:```select columns from library.dataset where column1 = value```. While it's of course an option to just manually write SQL queries, this gets very fast, very cumbersome when specifying many conditions (I also tend to sneak in typos).

The following `python` function can take different arguments--such as column names or SIC codes--and automatically builds the query for a specific dataset (the only required argyment).

```python
import wrds
import pandas as pd
```


```python
def build_query(dataset, **kwargs):
    """Build Query for WRDS database.
        Args:
            dataset (str): dataset
            start (str): Start of time period with format YYYY-MM-DD, default to None
            end (str): End of time period with format YYYY-MM-DD, default to None
            columns (str or list): Filter of one or a list of columns, default to None
            sic (str or list): Filter of one or a list of SIC, default to None
            sich (str or list): Filter of one or a list of historical SIC, default to None
            gvkey (str or list):  Filter of one or a list of Global Company Key, default to None
            tic (str or list):  Filter of one or a list of ticker symbols, default to None
            cusip (str or list):  Filter of one or a list of CUSIP symbols, default to None
            limit (int): Return only a number of return records, defualt to None
        Returns:
            pandas.DataFrame: WRDS query results
    """
    columns = kwargs.get("columns", None)
    sic = kwargs.get("sic", None)
    sich = kwargs.get("sich", None)
    start = kwargs.get("start", None)
    end = kwargs.get("end", None)
    gvkey = kwargs.get("gvkey", None)
    tic = kwargs.get("tic", None)
    cusip = kwargs.get("cusip", None)
    limit = kwargs.get("limit", None)


    if columns is not None:
        columns_filter = ",".join([x for x in columns])
    else:
        columns_filter = "*"

    date_filter = ""
    if start and end is not None:
        date_filter = "datadate between {} and {}".format("'" + start + "'" , "'" + end + "'")

    sic_filter = ""
    if sic is not None:
        sic_filter = "sic in ({}) ".format(
            ",".join(["'" + x + "'" for x in sic])
        )

    sich_filter = ""
    if sich is not None:
        sich_filter = "sich in ({}) ".format(
            ",".join(["'" + x + "'" for x in sich])
        )

    gvkey_filter = ""
    if gvkey is not None:
        gvkey_filter = "gvkey in ({}) ".format(
            ",".join(["'" + x + "'" for x in gvkey])
        )

    tic_filter = ""
    if tic is not None:
        tic_filter = "tic in ({}) ".format(
            ",".join(["'" + x + "'" for x in tic])
        )

    cusip_filter = ""
    if cusip is not None:
        cusip_filter = "cusip in ({}) ".format(
            ",".join(["'" + x + "'" for x in cusip])
        )

    # FIRST INSTANCE WITHOUT AND THEN SEQUENTIALLY ADD FILTERS (if non empty)
    filters = [date_filter, cusip_filter, tic_filter, gvkey_filter, sic_filter]
    filters_list = ""
    if any(filters):
        filters_list = " and ".join([x for x in filters if x != ""])
        filters_list = "where " + filters_list

    limit_number = ""
    if limit:
        limit_number += "LIMIT {} ".format(limit)

    cmd = (
        "select "
        + columns_filter
        + " from {} ".format(dataset)
        + filters_list
        + limit_number
        )

    print(cmd)

    return(cmd)
```

Now let's see the function in action. The example case uses the Compustat database (`comp`). Let's assume we're interested in companies in specific industrial sectors and we know their SIC codes. We would now like to retrieve their financial fundamentals.

Since `comp.funda` does not allow us to filter based SIC codes directly, we first extract the codes of the relevant companies from the `comp.names` database.


```python
sic_codes = ['1011', '1021', '1031', '1041', '1044', '1061', '1081', '1094']

# construct query
query = build_query(dataset ="comp.names",
                    sic = sic_codes
                   )
```

    select * from comp.names where sic in ('1011','1021','1031','1041','1044','1061','1081','1094')



```python
# let's send the query to WRDS

# establish connection
db = wrds.Connection(wrds_username = 'davidkre')
```

    Loading library list...
    Done



```python
# qery database
df = db.raw_sql(query)
```

At this point the function "unfolds its power" and makes your life a lot easier.

To illustrate this and make things more interesting, let's assume that we do not want to download all of the variables but just the ones of interest. Moreover, we are only interested in a particular time period.  

```python
# get identifier of companies from query result
gvkeys = df["gvkey"].astype(str).to_list()


columns = ["gvkey","datadate","fyear","indfmt","consol",
           "popsrc","datafmt","tic","cusip", "conm","revt"]

# use gvkeys for new query
query = build_query(dataset ="comp.funda",
                    columns = columns,
                    gvkey = gvkeys,
                    start = '2000-01-01',
                    end = '2020-01-01',
                   )
```

    select gvkey,datadate,fyear,indfmt,consol,popsrc,datafmt,tic,cusip,conm,revt from comp.funda where datadate between '2000-01-01' and '2020-01-01' and gvkey in ('002127','003104','003808','009725','010171','010174','010910','013036','013400','014136','017005','033113','062038','066405','105464','105572','107248','108326','108768','156014','160849','165672','171083','175060','179566','186093','186778','187597')



```python
# qery database
df = db.raw_sql(query)
df.head()
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
      <th>gvkey</th>
      <th>datadate</th>
      <th>fyear</th>
      <th>indfmt</th>
      <th>consol</th>
      <th>popsrc</th>
      <th>datafmt</th>
      <th>tic</th>
      <th>cusip</th>
      <th>conm</th>
      <th>revt</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>010174</td>
      <td>2000-12-31</td>
      <td>2000.0</td>
      <td>INDL</td>
      <td>C</td>
      <td>D</td>
      <td>STD</td>
      <td>SSMR</td>
      <td>867833600</td>
      <td>SUNSHINEMINING&amp;REFINING</td>
      <td>22.927</td>
    </tr>
    <tr>
      <th>1</th>
      <td>010174</td>
      <td>2000-12-31</td>
      <td>2000.0</td>
      <td>INDL</td>
      <td>C</td>
      <td>D</td>
      <td>SUMM_STD</td>
      <td>SSMR</td>
      <td>867833600</td>
      <td>SUNSHINEMINING&amp;REFINING</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>013400</td>
      <td>2000-01-31</td>
      <td>1999.0</td>
      <td>INDL</td>
      <td>C</td>
      <td>D</td>
      <td>STD</td>
      <td>ASM</td>
      <td>053906103</td>
      <td>AVINOSILVER&amp;GOLDMINSLTD</td>
      <td>0.000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>013400</td>
      <td>2000-01-31</td>
      <td>1999.0</td>
      <td>INDL</td>
      <td>C</td>
      <td>D</td>
      <td>SUMM_STD</td>
      <td>ASM</td>
      <td>053906103</td>
      <td>AVINOSILVER&amp;GOLDMINSLTD</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>013400</td>
      <td>2001-01-31</td>
      <td>2000.0</td>
      <td>INDL</td>
      <td>C</td>
      <td>D</td>
      <td>STD</td>
      <td>ASM</td>
      <td>053906103</td>
      <td>AVINOSILVER&amp;GOLDMINSLTD</td>
      <td>0.000</td>
    </tr>
  </tbody>
</table>
</div>


```python
# don't forget to close your session if you're done ;)
db.close()
```
