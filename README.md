Using PySpark we can process data from Hadoop HDFS, AWS S3, and many file systems. PySpark also is used to process real-time data using Streaming and Kafka.

All Spark examples provided in this PySpark (Spark with Python) tutorial is basic, simple, and easy to practice for beginners who are enthusiastic to learn PySpark and advance their career in BigData and Machine Learning.

## Features of PySpark
<ul>In-memory computation</ul>
<ul>Distributed processing using parallelize</ul>
<ul>Can be used with many cluster managers (Spark, Yarn, Mesos e.t.c)</ul>
<ul>Fault-tolerant</ul>
<ul>Immutable</ul></ul>
<ul>Lazy evaluation</ul>
<ul>Cache & persistence</ul>
<ul>Inbuild-optimization when using DataFrames</ul>
<ul>Supports ANSI SQL</ul>

# PySpark Quick Reference
A quick reference guide to the most commonly used patterns and functions in PySpark SQL

### Read CSV file into DataFrame with schema and delimited as comma
```

df = spark.read.option(header='True', inferSchema='True',delimiter=',').csv("/tmp/resources/sales.csv")

```


### Easily reference these as F.func() and T.type()
```
from pyspark.sql import functions as F, types as T
```
## Common Operation
```
### Filter on equals condition

df = df.filter(df.is_adult == 'Y')

### Filter on >, <, >=, <= condition

df = df.filter(df.age > 25)

### Sort results

df = df.orderBy(df.age.asc()))  
df = df.orderBy(df.age.desc()))


### Multiple conditions require parentheses around each condition

df = df.filter((df.age > 25) & (df.is_adult == 'Y'))


### Compare against a list of allowed values

df = df.filter(col('age').isin([3, 4, 7]))
```
## Joins
```
### Left join in another dataset

df = df.join(person_lookup_table, 'person_id', 'left')

### Match on different columns in left & right datasets
df = df.join(other_table, df.id == other_table.person_id, 'left')

### Match on multiple columns
df = df.join(other_table, ['first_name', 'last_name'], 'left')

### Useful for one-liner lookup code joins if you have a bunch
def lookup_and_replace(df1, df2, df1_key, df2_key, df2_value):
    return (
        df1
        .join(df2[[df2_key, df2_value]], df1[df1_key] == df2[df2_key], 'left')
        .withColumn(df1_key, F.coalesce(F.col(df2_value), F.col(df1_key)))
        .drop(df2_key)
        .drop(df2_value)
    )

df = lookup_and_replace(people, pay_codes, id, pay_code_id, pay_code_desc)
```

## Column Operations
```
# Add a new static column
df = df.withColumn('status', F.lit('PASS'))

# Construct a new dynamic column
df = df.withColumn('full_name', F.when(
    (df.fname.isNotNull() & df.lname.isNotNull()), F.concat(df.fname, df.lname)
).otherwise(F.lit('N/A'))

# Pick which columns to keep, optionally rename some
df = df.select(
    'name',
    'age',
    F.col('dob').alias('date_of_birth'),
)
```
## Casting & Coalescing Null Values & Duplicates
```
# Cast a column to a different type
df = df.withColumn('price', df.price.cast(T.DoubleType()))

# Replace all nulls with a specific value
df = df.fillna({
    'first_name': 'Sundar',
    'age': 18,
})

# Take the first value that is not null
df = df.withColumn('last_name', F.coalesce(df.last_name, df.surname, F.lit('N/A')))

# Drop duplicate rows in a dataset (distinct)
df = df.dropDuplicates()

# Drop duplicate rows, but consider only specific columns
df = df.dropDuplicates(['name', 'height'])

# Replace empty strings with null (leave out subset keyword arg to replace in all columns)
df = df.replace({"": None}, subset=["name"])

# Convert Python/PySpark/NumPy NaN operator to null
df = df.replace(float("nan"), None)


# Remove columns
df = df.drop('mod_dt', 'mod_username')

# Rename a column
df = df.withColumnRenamed('dob', 'date_of_birth')

# Keep all the columns which also occur in another dataset
df = df.select(*(F.col(c) for c in df2.columns))

# Batch Rename/Clean Columns
for col in df.columns:
    df = df.withColumnRenamed(col, col.lower().replace(' ', '_').replace('-', '_'))
```
## String Operations
### String Filters
```
# Contains - col.contains(string)
df = df.filter(df.name.contains('o'))

# Starts With - col.startswith(string)
df = df.filter(df.name.startswith('Al'))

# Ends With - col.endswith(string)
df = df.filter(df.name.endswith('ice'))

# Is Null - col.isNull()
df = df.filter(df.is_adult.isNull())

# Is Not Null - col.isNotNull()
df = df.filter(df.first_name.isNotNull())

# Like - col.like(string_with_sql_wildcards)
df = df.filter(df.name.like('Al%'))

# Regex Like - col.rlike(regex)
df = df.filter(df.name.rlike('[A-Z]*ice$'))

# Is In List - col.isin(*cols)
df = df.filter(df.name.isin('Bob', 'Mike'))
```
### String Functions
```
# Substring - col.substr(startPos, length)
df = df.withColumn('short_id', df.id.substr(0, 10))

# Trim - F.trim(col)
df = df.withColumn('name', F.trim(df.name))

# Left Pad - F.lpad(col, len, pad)
# Right Pad - F.rpad(col, len, pad)
df = df.withColumn('id', F.lpad('id', 4, '0'))

# Left Trim - F.ltrim(col)
# Right Trim - F.rtrim(col)
df = df.withColumn('id', F.ltrim('id'))

# Concatenate - F.concat(*cols)
df = df.withColumn('full_name', F.concat('fname', F.lit(' '), 'lname'))

# Concatenate with Separator/Delimiter - F.concat_ws(delimiter, *cols)
df = df.withColumn('full_name', F.concat_ws('-', 'fname', 'lname'))

# Regex Replace - F.regexp_replace(str, pattern, replacement)[source]
df = df.withColumn('id', F.regexp_replace(id, '0F1(.*)', '1F1-$1'))

# Regex Extract - F.regexp_extract(str, pattern, idx)
df = df.withColumn('id', F.regexp_extract(id, '[0-9]*', 0))
```

## Number Operations
```
# Round - F.round(col, scale=0)
df = df.withColumn('price', F.round('price', 0))

# Floor - F.floor(col)
df = df.withColumn('price', F.floor('price'))

# Ceiling - F.ceil(col)
df = df.withColumn('price', F.ceil('price'))

# Absolute Value - F.abs(col)
df = df.withColumn('price', F.abs('price'))

# X raised to power Y – F.pow(x, y)
df = df.withColumn('exponential_growth', F.pow('x', 'y'))

# Select smallest value out of multiple columns – F.least(*cols)
df = df.withColumn('least', F.least('subtotal', 'total'))

# Select largest value out of multiple columns – F.greatest(*cols)
df = df.withColumn('greatest', F.greatest('subtotal', 'total'))
```

## Date & Timestamp Operations
```
# Convert a string of known format to a date (excludes time information)
df = df.withColumn('date_of_birth', F.to_date('date_of_birth', 'yyyy-MM-dd'))

# Convert a string of known format to a timestamp (includes time information)
df = df.withColumn('time_of_birth', F.to_timestamp('time_of_birth', 'yyyy-MM-dd HH:mm:ss'))

# Get year from date:       F.year(col)
# Get month from date:      F.month(col)
# Get day from date:        F.dayofmonth(col)
# Get hour from date:       F.hour(col)
# Get minute from date:     F.minute(col)
# Get second from date:     F.second(col)
df = df.filter(F.year('date_of_birth') == F.lit('2017'))

# Add & subtract days
df = df.withColumn('three_days_after', F.date_add('date_of_birth', 3))
df = df.withColumn('three_days_before', F.date_sub('date_of_birth', 3))

# Add & Subtract months
df = df.withColumn('next_month', F.add_month('date_of_birth', 1))

# Get number of days between two dates
df = df.withColumn('days_between', F.datediff('start', 'end'))

# Get number of months between two dates
df = df.withColumn('months_between', F.months_between('start', 'end'))

# Keep only rows where date_of_birth is between 2017-05-10 and 2018-07-21
df = df.filter(
    (F.col('date_of_birth') >= F.lit('2017-05-10')) &
    (F.col('date_of_birth') <= F.lit('2018-07-21'))
)
```

## Array Operations
```
# Column Array - F.array(*cols)
df = df.withColumn('full_name', F.array('fname', 'lname'))

# Empty Array - F.array(*cols)
df = df.withColumn('empty_array_column', F.array([]))

# Array Size/Length – F.size(col)
df = df.withColumn('array_length', F.size(F.col('my_array')))
```
## Aggregation Operations
```
# Row Count:                F.count()
# Sum of Rows in Group:     F.sum(*cols)
# Mean of Rows in Group:    F.mean(*cols)
# Max of Rows in Group:     F.max(*cols)
# Min of Rows in Group:     F.min(*cols)
# First Row in Group:       F.alias(*cols)
df = df.groupBy('gender').agg(F.max('age').alias('max_age_by_gender'))

# Collect a Set of all Rows in Group:       F.collect_set(col)
# Collect a List of all Rows in Group:      F.collect_list(col)
df = df.groupBy('age').agg(F.collect_set('name').alias('person_names'))

# Just take the lastest row for each combination (Window Functions)
from pyspark.sql import Window as W

window = W.partitionBy("first_name", "last_name").orderBy(F.desc("date"))
df = df.withColumn("row_number", F.row_number().over(window))
df = df.filter(F.col("row_number") == 1)
df = df.drop("row_number")
```

## Advanced Operations
### Repartitioning
```
# Repartition – df.repartition(num_output_partitions)
df = df.repartition(1)
```

### UDFs (User Defined Functions)
```
# Multiply each row's age column by two
times_two_udf = F.udf(lambda x: x * 2)
df = df.withColumn('age', times_two_udf(df.age))

# Randomly choose a value to use as a row's name
import random

random_name_udf = F.udf(lambda: random.choice(['Bob', 'Tom', 'Amy', 'Jenna']))
df = df.withColumn('name', random_name_udf())
```

## Window Functions
<table id="tablepress-43" class="tablepress tablepress-id-43">
<thead>
<tr class="row-1 odd">
<th class="column-1">Window Functions Usage &amp; Syntax</th><th class="column-2">PySpark Window Functions description</th>
</tr>
</thead>
<tbody>
<tr class="row-2 even">
<td class="column-1">row_number(): Column</td><td class="column-2">Returns a sequential number starting from 1 within a window partition</td>
</tr>
<tr class="row-3 odd">
<td class="column-1">rank(): Column</td><td class="column-2">Returns the rank of rows within a window partition, with gaps.</td>
</tr>
<tr class="row-4 even">
<td class="column-1">percent_rank(): Column</td><td class="column-2">Returns the percentile rank of rows within a window partition.</td>
</tr>
<tr class="row-5 odd">
<td class="column-1">dense_rank(): Column</td><td class="column-2">Returns the rank of rows within a window partition without any gaps. Where as Rank() returns rank with gaps.</td>
</tr>
<tr class="row-6 even">
<td class="column-1">ntile(n: Int): Column </td><td class="column-2">Returns the ntile id in a window partition</td>
</tr>
<tr class="row-7 odd">
<td class="column-1">cume_dist(): Column</td><td class="column-2">Returns the cumulative distribution of values within a window partition</td>
</tr>
<tr class="row-8 even">
<td class="column-1">lag(e: Column, offset: Int): Column<br />
lag(columnName: String, offset: Int): Column<br />
lag(columnName: String, offset: Int, defaultValue: Any): Column</td><td class="column-2">returns the value that is `offset` rows before the current row, and `null` if there is less than `offset` rows before the current row.</td>
</tr>
<tr class="row-9 odd">
<td class="column-1">lead(columnName: String, offset: Int): Column<br />
lead(columnName: String, offset: Int): Column<br />
lead(columnName: String, offset: Int, defaultValue: Any): Column</td><td class="column-2">returns the value that is `offset` rows after the current row, and `null` if there is less than `offset` rows after the current row.</td>
</tr>
</tbody>
</table>

### row_number Window Function
```

from pyspark.sql.window import Window
from pyspark.sql.functions import row_number
windowSpec  = Window.partitionBy("department").orderBy("salary")

df.withColumn("row_number",row_number().over(windowSpec)) \
    .show(truncate=False)
```





