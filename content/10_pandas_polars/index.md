+++
title = "Upgrading The Basic: Polars Edition"
date = 2024-03-29
authors = ["Amri Rasyidi"]
description = "An introduction to pandas alternative: polars"
draft = false

[taxonomies]
series = []
tags = ["data-science", "how-to", "python"]
+++

If you have some experience of doing data science stuff with python, chances are, you are familiar with pandas. Pandas is almost the holy grail for data scientists that use python.

In this post, I want to introduce an alternative (or rather a complement?) to pandas.

Enter [<b>polars</b>](https://pola-rs.github.io/polars/).

Let's get straight to the showcase, then some explanation later. Also, if you want to see the performance comparison of pandas vs 

# Get started
## Install, import, and read data
Install and import are quite straightforward...
```python
pip install polars
import polars as pl
```
If you set it to default, polars will utilize all the cpu cores (or thread, I don't really know ¯\_(ツ)_/¯) available on your machine. In normal situation, I would open other things (e.g. browser, spotify, and another browser) besides VSCode where I run my jupyter notebook. Polars consuming all the resource while it given a heavy task can be a problem, I've experience my machine bricked (blue screen or freezing) because of this.

To prevent that, you can limit the number of cores that will be used by polars.
```python
# get the number of available cpu
num_cores = multiprocessing.cpu_count()
# set it to half of the available cpu
os.environ["POLARS_NUM_THREADS"] = str(num_cores//2)
```
After you import the library, one of the first thing you can do with polars is to read your data. You can find the complete list of supported data format that can be read by polars in the [official documentation](https://pola-rs.github.io/polars/py-polars/html/reference/).

There are 2 'paradigms' to read a data using polars, 'eagerly' and 'lazyly'.
```python
df_pandas = pd.read_parquet(r"path\to\file.parquet")
df_eager = pl.read_parquet(r"path\to\file.parquet")
df_lazy = pl.scan_parquet(r"path\to\file.parquet")
```
I will touch on this eager and lazy concept briefly <a href="#what-about-the-lazy-thingy">later</a>.

## Most common syntaxes
After reading the data into memory, you may want to do manipulation or analysis. There are common syntaxes in polars that you will see quite often once you get into it.
1. `select`
    <br>Select is, as obvious as it is, used to select/slice <b>columns</b>. In the example below, the syntax only slice some columns from a dataframe, without altering anything.
    ```python
    # select all column using wildcard (*)
    df_eager.select(pl.col("*"))

    # select all except one
    df_eager.select([pl.exclude("column_1")])

    # select column_1 and 2 to be shown
    df_eager.select(
        pl.col('column_1'),
        pl.col('column_2')
    )

    # equivalent with
    df_eager[['column_1','column_2']]

    # pandas equivalent
    df_pandas[['column_1','column_2']]
    ```
    Somewhat, I actually prefer the first one with `pl.col()`. You will have to type more but the code looks more readable. And in polars, this is usually the case anyway.
1. `with_columns`
    <br>Use this to create/transform/assign columns. Let's say you want to create a new column from an available polars dataframe, you can to it like this.
    ```python
    # this doesn't alter the original dataframe
    df_eager.with_columns(
        # create a new col_1_min, value = min of column_1
        pl.col('column_1').min().alias('col_1_min'),
        # create a new col_2_int, value = integer version of column_2
        pl.col("column_2").cast(pl.Int64).alias('col_2_int')
    )
    # this one does (adding 2 new columns)
    df_eager = df_eager.with_columns(
        pl.col('column_1').min().alias('col_1_min'),
        pl.col("column_2").cast(pl.Int64).alias('col_2_int')
    )
    # pandas equivalent
    df_pandas = df_pandas.assign(
        col_1_min=df_pandas['column_1'].min(),
        col_2_int=df_pandas['column_2'].astype('int64')
    )
    ```

1. `filter`
    <br>This one is for filter/slice/query <b>rows</b>
    ```python
    # show all COLUMN, where ROW fulfill the criteria
    df_eager.filter(
        (pl.col('column_1')>=some_value) # column_1 more than or equal some_value
        & (~pl.col('column_2').is_null()) # column_2 is NOT null
    )

    # pandas equivalent
    df_pandas[(
        (df_pandas['column_1']>=some_value) # column_1 more than or equal some_value
        & (~df_pandas['column_2'].is_null()) # column_2 is NOT null
    )]
    ```
1. `join`
    <br>Join in SQL-like fashion.
    ```python
    # join df_eager to df_eager_2 (using inner join by default) on 'shared_column_name'
    df_eager.join(
        df_eager_2, on='shared_column_name'
    )
    # or with different column name
    df_eager.join(
        df_eager_2, left_on="column", right_on="column_2"
    )

    # pandas equivalent with the first one
    df_pandas.merge(df_eager_2, on='shared_column_name')
    ```
1. `group_by` and `agg`
    <br>Group dataframe <b>rows</b> then make aggregation to the groups
    ```python
    # group the data by column_1 and 2, then create new column by aggregation
    df_eager.group_by(
        ['column_1','column_2']
    ).agg(
        pl.col('column_3').n_unique().alias('col_3_nunique'), # number of unique instances per group
        pl.col('column_4').sum().alias('col_4_sum') # sum of 'column_4' per group
    )

    # pandas equivalent
    df_pandas.groupby(['column_1', 'column_2']).agg(
        col_3_nunique=('column_3', 'nunique'),  # number of unique instances per group
        col_4_sum=('column_4', 'sum')  # sum of 'column_4' per group
    ).reset_index()
    ```
1. `sort`
    <br>sort dataframe by column, ascending by default
    ```python
    df_eager.sort('booking_time')

    # pandas equivalent
    df_pandas.sort_values('booking_time')
    ```

## What about the lazy thingy?

From the documentation: 
_"In the eager API the query is executed immediately while in the lazy API the query is only evaluated once it is 'needed'"_

Simply said, preventing the evaluation until it is needed allows the query planner to performs various optimization, leading to faster and more efficient query execution.

# Why polars?

There are some reasons polars might be "an upgrade" from pandas. First, it is fast!  Polars utilizes all available cores on your machine and optimizes queries to reduce unneeded work/memory allocations. Polars can also handles datasets much larger than your available RAM.

There are also small things that I like about polars:
- Show data type. You don't have to execute 2 lines of codes to check the overview of the data and check the data type from each columns.
- Nice syntax (feels like sql). There is actually a way to use sql in polars. Is is also _feels_ consistent and predictable.
- Easy to learn. Know pandas = know polars 80%

# Closing thought

I think it is worth to learn polars. Heck, if I have to tell others what to learn to master data science with python, I would suggest pandas and polars (if not only polars).

